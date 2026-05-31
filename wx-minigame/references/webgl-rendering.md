# WebGL 离屏渲染详细参考

解决 iOS Canvas 2D 渐变色阶（banding）的 WebGL 方案。核心原则见 SKILL.md §7，本文档是其完整实现参考。

---

## 问题根因

iOS 设备上 Canvas 2D 的 `createLinearGradient` 在大面积渐变（如全屏背景）时出现明显色阶/色带，尤其深色系渐变。根因：Canvas 2D 渐变在 sRGB 空间做线性插值，且没有 dithering。增加 colorStop 数量能缓解但无法根治。

## 方案：WebGL 离屏 canvas + shader dithering

用 WebGL 离屏 canvas 渲染渐变，在 fragment shader 中做线性光空间插值 + 噪声抖动（±0.5 LSB），再 drawImage 到主 2D canvas。

### 关键 shader

```glsl
precision highp float;
varying vec2 v_uv;
uniform vec3 u_topColor;
uniform vec3 u_bottomColor;
uniform vec2 u_resolution;

vec3 srgbToLinear(vec3 c) {
  return mix(c / 12.92, pow((c + 0.055) / 1.055, vec3(2.4)), step(0.04045, c));
}
vec3 linearToSrgb(vec3 c) {
  return mix(c * 12.92, 1.055 * pow(c, vec3(1.0 / 2.4)) - 0.055, step(0.0031308, c));
}
float hash(vec2 p) {
  return fract(sin(dot(p, vec2(12.9898, 78.233))) * 43758.5453);
}

void main() {
  // 线性光空间插值（避免 sRGB 空间插值导致色带加重）
  vec3 topLin = srgbToLinear(u_topColor);
  vec3 botLin = srgbToLinear(u_bottomColor);
  vec3 color = mix(topLin, botLin, v_uv.y);
  color = linearToSrgb(color);

  // ±0.5 LSB 噪声抖动消除色阶
  color += (hash(gl_FragCoord.xy) - 0.5) / 255.0;

  gl_FragColor = vec4(clamp(color, 0.0, 1.0), 1.0);
}
```

---

## iOS WebGL 三大陷阱

### 1. preserveDrawingBuffer 必须为 true

iOS 上 WebGL drawing buffer 默认帧间清除。不设此项，渲染后 `drawImage` 或 `readPixels` 读到空数据。

```typescript
const gl = offCanvas.getContext('webgl', {
  preserveDrawingBuffer: true,  // iOS 必须
  antialias: false,             // 关闭抗锯齿，避免 readPixels 黑屏
});
```

### 2. 每帧必须重绑 GL 状态

iOS WebKit 可能帧间丢失 GL 状态（program、buffer binding、viewport）。每帧显式重绑：

```typescript
gl.useProgram(program);
gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
gl.enableVertexAttribArray(aPosition);
gl.vertexAttribPointer(aPosition, 2, gl.FLOAT, false, 0, 0);
gl.viewport(0, 0, width, height);
```

### 3. drawImage(WebGL → 2D) 跨 context 兼容

并非所有平台支持将 WebGL canvas 作为 `drawImage` source。Android 小游戏可能读到空数据。

**运行时自测**：

```typescript
function testDrawImage(glCanvas: any, gl: WebGLRenderingContext): boolean {
  gl.clearColor(1.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.flush();

  const test = createOffscreenCanvas(1, 1);
  const ctx2d = test.getContext('2d');
  ctx2d.drawImage(glCanvas, 0, 0, glCanvas.width, glCanvas.height, 0, 0, 1, 1);
  const pixel = ctx2d.getImageData(0, 0, 1, 1).data;
  return pixel[0] > 200 && pixel[1] < 50; // 读到红色说明可用
}
```

---

## 渲染模式三级回退

| 优先级 | 模式 | 条件 | 性能 |
|--------|------|------|------|
| 1 | GL_DIRECT | WebGL 可用 + drawImage 自测通过 | 最快 |
| 2 | GL_READPIXELS | WebGL 可用但 drawImage 不可用 | 较慢（每帧读全屏像素） |
| 3 | CANVAS_2D | WebGL 不可用 | 中等（多 colorStop 渐变） |

实践中 readPixels 每帧读全屏像素（~3.5MB @ 1080p）开销极大。drawImage 不可用时，直接回退 Canvas 2D + 多档 colorStop 更划算。

---

## Canvas 2D 回退优化

48 档 colorStop + 线性光空间插值缓解色阶：

```typescript
const STOPS = 48;
const grad = ctx.createLinearGradient(0, 0, 0, height);
for (let i = 0; i <= STOPS; i++) {
  const t = i / STOPS;
  const [r, g, b] = lerpInLinearSpace(topColor, bottomColor, t);
  grad.addColorStop(t, `rgb(${r},${g},${b})`);
}
```

---

## readPixels 内存复用

如果确实需要 readPixels 路径，避免每帧分配大 buffer：

```typescript
// 仅在尺寸变化时重新分配，帧间复用
if (w !== bufWidth || h !== bufHeight) {
  pixelBuf = new Uint8Array(w * h * 4);
  tmpRowBuf = new Uint8Array(w * 4); // 行翻转用
  bufWidth = w;
  bufHeight = h;
}
// readPixels 行序从下到上，需翻转
gl.readPixels(0, 0, w, h, gl.RGBA, gl.UNSIGNED_BYTE, pixelBuf);
```
