---
name: wx-minigame
description: 微信小游戏开发完整指南，覆盖 H5 到小游戏的全链路迁移适配。当用户提到微信小游戏、minigame、wx适配、H5移植、Canvas游戏迁移时必须使用此 skill。也适用于用户遇到小游戏特有问题的场景，例如"小游戏黑屏"、"iOS真机没声音"、"模拟器正常真机不行"、"Can't find variable setTimeout"、"weapp-adapter"、"wx.createCanvas"、"胶囊按钮遮挡"、"触摸事件不生效"、"InnerAudioContext"、"wx Storage"等。即使用户没有明确说"小游戏"，只要涉及 GameGlobal、wx.onTouchStart、wx.createInnerAudioContext 等微信小游戏 API，也应触发此 skill。也覆盖小游戏中使用 WebGL 的场景，如"渐变色阶""banding""preserveDrawingBuffer""drawImage WebGL canvas"等。
user_invocable: true
---

# 微信小游戏开发与 H5 迁移指南

将 H5 Canvas 应用迁移到微信小游戏平台的完整适配参考。

## 如何使用本指南

- **编写小游戏新功能时**：先查§1 速查表确认 API 差异，再参照对应章节编写适配代码
- **移植已有 H5 功能时**：直接走§16 迁移检查清单，逐项对照
- **排查真机问题时**：查§15 常见报错速查表，定位已知陷阱
- **实现触摸滚动时**：§11 有核心原则，详细参数见 `references/scroll-physics.md`

---

## 1. 运行环境差异速查表

| 能力 | H5 浏览器 | 微信小游戏 | 适配方式 |
|------|----------|-----------|---------|
| 全局变量 (setTimeout, window, document…) | 原生可用 | ES 模块作用域不可用 | weapp-adapter 注入 |
| Canvas 获取 | `document.getElementById()` | `wx.createCanvas()` 首次返回主屏画布 | 平台适配层 |
| 触摸事件 | `canvas.addEventListener()` | `wx.onTouchStart()` 全局 API | 平台分支 |
| 存储 | `localStorage` | `wx.getStorageSync/setStorageSync` | 统一封装 |
| 音频 | Web Audio API | `wx.createInnerAudioContext()` | 双模引擎 |
| 图片加载 | `new Image()` | `wx.createImage()` | 适配层 |
| ctx.filter | 支持 blur 等 | 不支持 | 运行时检测降级 |
| DOM 滚动 | 原生可用 | 不存在 | Canvas 手动实现 |
| 分享 | Web Share API（有限） | `wx.shareAppMessage()` | 平台封装 |
| 离屏 Canvas | `document.createElement('canvas')` | `wx.createCanvas()`（第 2 次起） | 适配层 |
| 导出图片 | `canvas.toDataURL()` | `canvas.toTempFilePath()` | Promise 封装 |

---

## 2. weapp-adapter 模式

微信小游戏运行在 JavaScriptCore（iOS）/ V8（Android）上，没有浏览器环境。`setTimeout`、`performance`、`window`、`document` 等全局变量在 `GameGlobal` 上存在，但 **ES 模块作用域无法直接访问**。第三方库（如 Matter.js、PixiJS）内部依赖这些变量会崩溃。

### 标准做法

创建 `weapp-adapter.js`，在游戏代码加载前将浏览器 API 从 `GameGlobal` 注入到 `globalThis`。

**加载顺序（关键）**：
```javascript
// minigame/game.js — 入口文件
import './weapp-adapter.js';   // 必须第一个导入
import './dist/main.js';       // 游戏代码
```

### 注入清单

| 类别 | 注入内容 | 来源 |
|------|---------|------|
| 定时器 | setTimeout, clearTimeout, setInterval, clearInterval, requestAnimationFrame, cancelAnimationFrame | GameGlobal → globalThis |
| 性能计时 | performance.now() | wx.getPerformance() 或 Date.now() 降级 |
| 主 Canvas | GameGlobal.__mainCanvas | wx.createCanvas() 首次调用 |
| window | 指向 GameGlobal | — |
| document | 最小实现（createElement, getElementById 等） | 手工构造 |
| Image | 包装 wx.createImage() | — |
| HTMLCanvasElement | 空构造函数 | 第三方库 instanceof 检查需要 |
| HTMLElement | 空构造函数 | — |
| navigator | userAgent/platform/language | wx.getSystemInfoSync() |
| XMLHttpRequest | 空构造函数 | 防止第三方库崩溃 |

### 注意事项

- 使用 `GameGlobal.__adapterInjected` 标记防止重复注入
- 所有注入带 `typeof === 'undefined'` 守卫，不覆盖已存在的原生实现
- 业务代码中可额外封装 `safeSetTimeout` / `safeNow` 作为双重保险

---

## 3. Canvas 双轨处理

### 主 Canvas 获取

`wx.createCanvas()` 的行为：
- **首次调用**：返回与屏幕等大的主屏 Canvas（唯一可见画布）
- **后续调用**：返回离屏 Canvas

如果 adapter 先调用了 `wx.createCanvas()`，游戏代码再调用时拿到的是离屏画布，导致**黑屏**。

**解决方案** — 通过 `GameGlobal.__mainCanvas` 共享：

```typescript
// weapp-adapter.js 中
const mainCanvas = wx.createCanvas();
GameGlobal.__mainCanvas = mainCanvas;

// 游戏代码中获取主 Canvas
function getMainCanvas(): HTMLCanvasElement {
  const g = typeof GameGlobal !== 'undefined' ? (GameGlobal as any) : null;
  return g?.__mainCanvas ?? wx.createCanvas();
}
```

### 离屏 Canvas

需要离屏画布（如截图、分享卡图）时，调用 `wx.createCanvas()`（非首次调用返回离屏画布）：

```typescript
function createOffscreenCanvas(w: number, h: number): HTMLCanvasElement {
  if (typeof wx !== 'undefined') {
    const c = wx.createCanvas();
    c.width = w;
    c.height = h;
    return c;
  }
  const c = document.createElement('canvas');
  c.width = w;
  c.height = h;
  return c;
}
```

---

## 4. 触摸事件适配

### 问题

小游戏的 Canvas 对象没有 `addEventListener` 方法。

### 解决方案

通过 `typeof wx !== 'undefined'` 检测平台，分支注册：

| 平台 | 注册方式 | 销毁方式 |
|------|---------|---------|
| 小游戏 | `wx.onTouchStart/Move/End/Cancel` | `wx.offTouchXxx` |
| H5 现代浏览器 | `canvas.addEventListener('pointerXxx')` | `removeEventListener` |
| H5 兼容 | `canvas.addEventListener('touchXxx')` | `removeEventListener` |

### 坐标转换

- **小游戏**：touch 事件坐标是逻辑像素（已考虑 DPR），与 Canvas 坐标一致，**无需额外转换**
- **H5**：需要减去 `canvas.getBoundingClientRect()` 偏移

```typescript
// 小游戏
const x = touch.clientX;
const y = touch.clientY;

// H5
const rect = canvas.getBoundingClientRect();
const x = touch.clientX - rect.left;
const y = touch.clientY - rect.top;
```

---

## 5. RAF 时间基准（iOS 真机陷阱）

### 问题

在 iOS 真机上，`performance.now()`（polyfill）和 `requestAnimationFrame` 回调参数 `now` 使用**不同的时间基准**。如果用 `performance.now()` 初始化 `lastTime`，第一帧的 `dt` 可能产生**巨大的负值或正值**，导致动画跳帧、物理系统异常。

**注意：微信开发者工具中不复现，仅 iOS 真机出现。**

### 解决方案

`lastTime` 从首帧 RAF 回调参数获取，不混用外部时间源：

```typescript
class GameLoop {
  private lastTime = -1; // 哨兵值

  start(): void {
    this.lastTime = -1;
    requestAnimationFrame(this.tick);
  }

  private tick = (now: number): void => {
    if (this.lastTime < 0) {
      this.lastTime = now; // 用 RAF 自身时间戳初始化
    }
    const dt = Math.min(now - this.lastTime, 50); // dt 上限防爆
    this.lastTime = now;
    this.onUpdate(dt, now);
    requestAnimationFrame(this.tick);
  };
}
```

**原则**：GameLoop 内部时间全部来自 RAF 回调参数，不混用 `performance.now()` / `Date.now()`。

---

## 6. iOS Canvas 渲染陷阱

### 低 alpha 残影

iOS Canvas 对极低 alpha 值（如 0.001）仍会渲染抗锯齿边缘像素，导致已消失的元素偶尔闪烁残影。

**解决方案**：
1. **渲染端过滤**：`displayAlpha < 0.01` 时跳过渲染（核心，所有场景适用）
2. **状态端清零**：元素进入"已销毁"状态时强制 `displayAlpha = 0, scale = 0`
3. **物理端冻结**（使用物理引擎时）：批量销毁时冻结物理体，防止动画期间被推动产生位移

```typescript
// 渲染时跳过近乎透明的元素
if (element.displayAlpha < 0.01) continue;

// 销毁时强制清零视觉属性
element.displayAlpha = 0;
element.scaleX = 0;
element.scaleY = 0;
```

### ctx.filter 不支持

小游戏 Canvas 不支持 `ctx.filter = 'blur(...)'`，直接使用会静默失败。

**运行时检测一次，缓存结果**：

```typescript
function supportsFilter(ctx: CanvasRenderingContext2D): boolean {
  try {
    ctx.filter = 'blur(1px)';
    const ok = ctx.filter === 'blur(1px)';
    ctx.filter = 'none';
    return ok;
  } catch {
    return false;
  }
}
```

不支持时降级为半透明纯色遮罩代替毛玻璃效果。

---

## 7. WebGL 离屏渲染（解决 iOS 渐变色阶）

iOS 上 Canvas 2D 大面积渐变会出现明显色阶（banding），因为渐变在 sRGB 空间插值且无 dithering。解决方案：WebGL 离屏 canvas 做线性光空间插值 + ±0.5 LSB 噪声抖动，再 drawImage 到主 2D canvas。

**iOS WebGL 三个必须注意的陷阱**：

1. **`preserveDrawingBuffer: true`** — iOS drawing buffer 默认帧间清除，不设此项 drawImage/readPixels 读到空数据
2. **每帧重绑 GL 状态** — iOS WebKit 帧间可能丢失 program/buffer/viewport，每帧显式重绑
3. **drawImage 跨 context 自测** — Android 小游戏可能不支持 WebGL canvas 作为 drawImage source，需运行时自测 + 三级回退（GL_DIRECT → GL_READPIXELS → CANVAS_2D）

实践中 readPixels 每帧读全屏像素开销极大，drawImage 不可用时直接回退 Canvas 2D（48 档 colorStop + 线性光空间插值）更划算。

完整 shader 代码、自测函数、回退策略和 readPixels 内存复用见 `references/webgl-rendering.md`。

---

## 8. 音频系统

### H5 路径

```
Web Audio API: AudioContext → fetch → decodeAudioData → BufferSource + GainNode
文件路径: /audio/xxx.mp3（绝对路径，Vite 静态服务）
```

### 小游戏路径

```
wx.createInnerAudioContext() 音频池，每个文件 N 个实例轮换
文件路径: audio/xxx.mp3（相对路径，minigame/audio/ 目录下）
实例属性: src, volume, playbackRate
播放前 seek(0) 重置位置
```

### iOS 静音模式

使用 `wx.setInnerAudioOption` 全局设置（SDK 2.3.0+）：

```typescript
// 在创建音频实例前调用一次
wx.setInnerAudioOption({ obeyMuteSwitch: false });
```

**注意**：实例属性 `audio.obeyMuteSwitch` 从 SDK 2.3.0 起已废弃，不再生效。必须通过 `wx.setInnerAudioOption` 统一设置。

### 注意事项

- 音频文件必须放在 `minigame/audio/` 目录下，路径为相对路径
- 音频池大小过小会导致快速连续触发时音效丢失

---

## 9. 平台 API 封装

多个 H5 API 在小游戏中有对应替代，统一封装模式：`typeof wx !== 'undefined'` 分支。

| 功能 | H5 API | 小游戏 API | 注意 |
|------|--------|-----------|------|
| 存储读 | `localStorage.getItem(key)` | `wx.getStorageSync(key)` | try-catch 包裹 |
| 存储写 | `localStorage.setItem(key, val)` | `wx.setStorageSync(key, val)` | try-catch 包裹 |
| 图片导出 | `canvas.toDataURL('image/png')` | `canvas.toTempFilePath({success,fail})` | 回调→Promise |
| 网络请求 | `fetch(url)` | `wx.request({url, success, fail})` | 需配置域名白名单 |
| 图片创建 | `new Image()` | `wx.createImage()` | — |

封装示例（存储）：

```typescript
const isWx = typeof wx !== 'undefined' && typeof wx.getStorageSync === 'function';

export function getItem(key: string): string | null {
  if (isWx) {
    try { return wx.getStorageSync(key) || null; }
    catch { return null; }
  }
  return localStorage.getItem(key);
}
```

其余 API 按同样模式封装。`wx.request` 需在微信后台配置合法域名白名单（开发时可勾选"不校验合法域名"）。

---

## 11. 安全区域与胶囊按钮

### 安全区域

**禁止硬编码**，动态获取：

```typescript
function getSafeArea() {
  if (typeof wx !== 'undefined') {
    const info = wx.getSystemInfoSync();
    const menu = wx.getMenuButtonBoundingClientRect();
    return {
      top: info.statusBarHeight,    // 刘海屏 44~59px
      menuBottom: menu.bottom,      // 约 58px
    };
  }
  return { top: 20, menuBottom: 0 };
}
```

### 胶囊按钮避让

微信小游戏右上角有不可移除的胶囊按钮，所有右上角 UI 必须避让：

```typescript
const { top: SAFE_AREA_TOP, menuBottom: MENU_BOTTOM } = getSafeArea();
const gap = 8;

// Y 起始位置
const entryTop = Math.max(SAFE_AREA_TOP + gap, MENU_BOTTOM + gap);

// 弹窗顶部
const panelTop = Math.max(SAFE_AREA_TOP, MENU_BOTTOM + 8);
```

---

## 12. Canvas 触摸滚动（惯性 + 橡皮筋完整方案）

小游戏没有 DOM，不存在原生滚动。列表面板、图鉴等需要手动实现触摸滚动 + 惯性。本节基于 woodfish 项目实际验证的实现。

### 架构分层

```
UILayer.handleScroll / startMomentum / stopMomentum  ← 接口层（每个可滚动层实现）
        ↓ 委托
ScrollContainer                                      ← 物理层（偏移量 + 惯性 + 弹簧）
        ↑ 驱动
BaseInputHandler（触摸状态机）                        ← 事件层（平台无关，两端复用）
```

### ScrollContainer：物理参数

```typescript
// 惯性衰减：每 16ms 衰减约 1.5%，~1.5 秒停止（速度 3 px/ms）
function decayVelocity(v: number, dt: number): number {
  return v * Math.pow(0.985, dt / 16)
}
// 停止阈值
if (Math.abs(velocity) < 0.01) stopMomentum()

// 橡皮筋阻力（顶/底越界时）：随过滚量线性增大，最大过滚 60px
const resistance = Math.max(0.05, 1 - over / 60)
offset += dy * resistance

// 弹回弹簧：每帧接近剩余距离的 22%（归一化到 16ms 帧率）
offset += (springTarget - offset) * Math.min(1, 0.22 * dt / 16)
// 收敛阈值：< 0.4px 直接对齐
```

```typescript
// ScrollContainer 完整接口
class ScrollContainer {
  setDimensions(contentHeight: number, viewportHeight: number): void
  handleScroll(dy: number): void     // dy 正 = 内容上移（手指下划对应负 dy，调用前取反）
  startMomentum(velocity: number): void  // 抬手时传入速度（px/ms）
  stopMomentum(): void
  update(dt: number): void           // 每帧调用，驱动惯性 + 弹簧
  get offset(): number               // 绘制时减去此值
  drawIndicator(ctx, x, panelY, panelH): void  // 右侧滚动条，内置淡入淡出
}
```

### BaseInputHandler：触摸状态机

```typescript
const SCROLL_THRESHOLD = 5        // px，超过才进入滚动模式
const VELOCITY_BLEND_NEW = 0.4    // EMA 权重
const VELOCITY_BLEND_OLD = 0.6
const WHEEL_SCALE = 0.003         // 滚轮 deltaY → px/ms

// 速度平滑（EMA，每次 touchMove 更新）
if (dt > 0) {
  const raw = frameDy / dt        // dt = Date.now() 差值，单位 ms
  velocity = velocity * 0.6 + raw * 0.4
}

// touchEnd：速度 clamp 后启动惯性（注意取反：负速度 = 内容向上）
scrollingLayer.startMomentum(clamp(-velocity, -3, 3))
```

**层锁定机制**：滑动开始后逆序遍历各层，第一个 `handleScroll()` 返回 `true` 的层被锁定，后续帧直接分发到该层，不再重新测试。

### UILayer 接口（可滚动层必须实现）

```typescript
// 必须实现以下 4 个方法，缺一不可
handleScroll(dy: number): boolean {
  if (!this.visible) return false   // 不可见必须返回 false
  this.scroll.handleScroll(dy)
  return true
}

startMomentum(velocity: number): boolean {
  if (!this.visible) return false   // 不可见必须返回 false
  this.scroll.startMomentum(velocity)
  return true
}

stopMomentum(): void {
  this.scroll.stopMomentum()
}

isPanelVisible(): boolean {
  return this.visible               // 供滚轮事件选层使用
}
```

### 渲染端（每帧）

```typescript
renderUI(ctx, w, h, dt) {
  if (!this.visible) return

  // 1. 每帧驱动物理（惯性 + 弹簧）
  this.scroll.setDimensions(contentHeight, viewportHeight)
  this.scroll.update(dt)

  const scrollY = this.scroll.offset

  // 2. 必须 clip 视口，否则过滚区内容溢出
  ctx.save()
  ctx.beginPath()
  ctx.rect(panelX, panelY, panelW, viewportH)
  ctx.clip()

  // 3. 绘制各行（y 坐标减去 scrollY）
  for (const item of items) {
    const y = panelY + item.row * rowH - scrollY
    if (y + rowH < panelY || y > panelY + viewportH) continue  // 视口外跳过
    // ...绘制...
  }

  ctx.restore()

  // 4. 绘制滚动指示器（内置 ~500ms 淡出）
  this.scroll.drawIndicator(ctx, panelX + panelW - 5, panelY, viewportH)
}
```

### 滚轮事件（桌面 H5）

滚轮事件找到最顶层满足 `isPanelVisible() !== false && handleScroll && startMomentum` 的层，累积速度（clamp 到 ±3）后调用 `startMomentum`。超过 200ms 无滚轮事件视为新一轮，重置累积速度。

### 常见坑

| 问题 | 原因 | 修复 |
|------|------|------|
| 滑动无效 | `handleScroll` 返回了 `false`（如未检查 visible） | 不可见时返回 false，可见时返回 true |
| 滚轮不响应 | 未实现 `isPanelVisible()` | 补充 `isPanelVisible(): boolean { return this.visible }` |
| 滚动穿透到下层 | 多层都返回 `true` | 确保只有当前可见层返回 true |
| 过滚区内容溢出 | 未 clip 视口 | `ctx.rect(panelX, panelY, panelW, viewportH); ctx.clip()` |
| 弹回动画抖动 | `setDimensions` 后 offset 被夹紧 | `setDimensions` 内直接 clamp，无弹簧，正常现象 |

---

## 13. 构建配置

### 包体大小限制

微信小游戏有严格的包体大小限制，这是构建阶段最常遇到的问题：

| 类型 | 限制 | 说明 |
|------|------|------|
| 主包 | **4MB** | game.js + adapter + 构建产物 + 必要资源 |
| 单个分包 | 8MB | 通过 `game.json` 的 `subpackages` 配置 |
| 总大小（主包+分包） | **20MB** | 超出需用远程资源 |

**常用优化手段**：
- 大图片/音频放 CDN，运行时下载到本地缓存（`wx.downloadFile` → `wx.getFileSystemManager`）
- 构建时 `minify: true` + `treeshake`
- 检查是否有意外打包的大依赖（`npx vite-bundle-visualizer`）

### Vite + ES 模块单文件输出

```typescript
// vite.config.wx.ts
export default defineConfig({
  build: {
    outDir: 'minigame/dist',
    lib: {
      entry: 'src/main.ts',
      formats: ['es'],
      fileName: 'main',
    },
    rollupOptions: {
      output: {
        inlineDynamicImports: true, // 打成一个文件
      },
    },
  },
});
```

### 目录结构

```
minigame/
├── game.js              # 入口（导入 adapter + main）
├── weapp-adapter.js     # 浏览器全局变量注入
├── game.json            # 游戏配置
├── project.config.json  # 微信 IDE 配置
├── dist/
│   └── main.js          # Vite 构建产物
├── audio/               # 音频文件
└── images/              # 图片资源
```

---

## 14. 真机调试要点

### 开发者工具 vs 真机差异

| 问题 | 开发者工具 | 真机 |
|------|-----------|------|
| 全局变量 | 部分可用（取决于工具版本） | ES 模块作用域严格隔离 |
| performance.now() 基准 | 与 RAF 一致 | 可能不一致（iOS） |
| Canvas 渲染精度 | 标准 | iOS 低 alpha 残影 |
| 静音模式 | 无影响 | obeyMuteSwitch 默认静音 |
| document 属性 | 真实 HTMLDocument（只读） | polyfill（可写） |
| 触摸事件时序 | 近乎同步 | 主线程繁忙时批量分发 |

**结论**：开发者工具能跑不等于真机能跑，必须在 iOS/Android 真机上测试。

### 常见报错速查

| 报错 | 原因 | 修复 |
|------|------|------|
| `Can't find variable: setTimeout` | ES 模块无法访问 GameGlobal 上的 timer | 确保 weapp-adapter.js 最先导入 |
| `Can't find variable: window` | 第三方库引用 window | adapter 注入 window = GameGlobal |
| `Cannot assign to read only property 'visibilityState'` | 开发者工具 document 是真实 DOM | try-catch 包裹赋值 |
| 黑屏无内容 | 主 Canvas 被 adapter 抢占 | 通过 GameGlobal.__mainCanvas 共享 |
| 动画首帧跳帧（iOS） | RAF 与 performance.now() 时间基准不一致 | lastTime 从 RAF 回调参数初始化 |
| 元素消失后残影闪烁（iOS） | 低 alpha Canvas 抗锯齿 | displayAlpha < 0.01 时跳过渲染 |
| 静音模式无声 | 默认遵守静音开关 | `wx.setInnerAudioOption({ obeyMuteSwitch: false })` |

---

## 15. 迁移检查清单

每次将 H5 功能移植到小游戏前，对照以下清单：

- [ ] **全局变量**：是否使用了 setTimeout、setInterval、fetch、XMLHttpRequest 等浏览器 API？检查 adapter 是否已注入
- [ ] **DOM 操作**：是否 createElement、getElementById、addEventListener？小游戏中可能不可用或行为不同
- [ ] **Canvas API**：是否使用了 ctx.filter、ctx.createPattern、ctx.createConicGradient 等高级 API？需检测可用性
- [ ] **事件监听**：新增 UI 交互是否通过平台适配层注册？是否需要支持触摸滚动？
- [ ] **资源路径**：音频、图片等资源是否放入 minigame/ 对应目录？路径是否使用相对路径？
- [ ] **安全区域**：新增 UI 元素是否动态获取安全区域？右上角是否避让胶囊按钮？
- [ ] **存储**：数据持久化是否使用统一封装的 getItem / setItem？
- [ ] **图片导出**：是否涉及 toDataURL？需改用 toTempFilePath 兼容
- [ ] **音频**：新音效是否在小游戏音频路径中正确配置？
- [ ] **WebGL**：如使用 WebGL 离屏渲染，是否设置 `preserveDrawingBuffer: true`？是否每帧重绑 GL 状态？drawImage 跨 context 是否自测通过？
- [ ] **时间基准**：GameLoop 是否仅使用 RAF 回调参数？触摸滚动是否传递 event.timeStamp？
- [ ] **真机测试**：iOS 和 Android 真机是否都测试通过？
