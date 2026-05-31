# Canvas 触摸滚动详细参考

小游戏没有 DOM 滚动，列表面板需要手动实现触摸滚动 + 惯性。本文档提供具体实现参数和踩坑记录。

核心原则见 SKILL.md §11，本文档是其补充。

---

## 速度估算 — 采样窗口法

参考 iScroll/Hammer.js 的做法：

1. 记录最近 ~10 个触摸采样 `{ cumulativeDy, event.timeStamp }`
2. 手指抬起时，取最近 100ms 窗口内的首末采样计算平均速度
3. 陈旧阈值 100ms：末采样距 touchEnd 的 `event.timeStamp` > 100ms → 视为手指已停顿，速度归零

```typescript
interface Sample {
  cumulativeDy: number;
  timestamp: number; // event.timeStamp
}

class ScrollPhysics {
  private samples: Sample[] = [];
  private readonly MAX_SAMPLES = 10;
  private readonly STALE_THRESHOLD = 100; // ms
  private readonly VELOCITY_WINDOW = 100; // ms

  addSample(cumulativeDy: number, eventTs: number): void {
    this.samples.push({ cumulativeDy, timestamp: eventTs });
    if (this.samples.length > this.MAX_SAMPLES) {
      this.samples.shift();
    }
  }

  estimateVelocity(endTs: number): number {
    if (this.samples.length < 2) return 0;

    const last = this.samples[this.samples.length - 1];
    // 陈旧检测：手指已停顿
    if (endTs - last.timestamp > this.STALE_THRESHOLD) return 0;

    // 在窗口内找最早的采样
    const windowStart = last.timestamp - this.VELOCITY_WINDOW;
    let first = last;
    for (const s of this.samples) {
      if (s.timestamp >= windowStart) {
        first = s;
        break;
      }
    }

    const dt = last.timestamp - first.timestamp;
    if (dt < 1) return 0;
    return (last.cumulativeDy - first.cumulativeDy) / dt; // px/ms
  }
}
```

### 为什么不用指数平滑

指数平滑（`v = 0.6 * oldV + 0.4 * newV`）在快速短甩（3-5 次 move 事件）时，速度只能达到实际值的 40-60%，容易低于启动阈值导致无惯性。采样窗口法直接取首末差值，不受历史权重影响。

---

## 惯性减速 — iOS 风格指数衰减

```typescript
private readonly DECEL = 0.998;     // 每 ms 衰减率
private readonly STOP_THRESHOLD = 0.02; // px/ms

update(dt: number): void {
  if (!this.isMomentumActive) return;
  this.velocity *= Math.pow(this.DECEL, dt);
  this.offset += this.velocity * dt;

  if (Math.abs(this.velocity) < this.STOP_THRESHOLD) {
    this.velocity = 0;
    this.isMomentumActive = false;
  }
}
```

### 参数选择

| 参数 | 值 | 效果 |
|------|-----|------|
| 衰减率 0.998^dt | dt=16 时约 0.968 | 自然手感，约 2-3 秒滑停 |
| 衰减率 0.985^(dt/16) | 同上约 0.985 | 太快，"一闪而过" |
| 衰减率 0.999^dt | dt=16 时约 0.984 | 偏滑腻，像冰面 |

---

## 橡皮筋越界回弹

### 拖动越界

当手指拖动超出内容边界时，施加阻力：

```typescript
onTouchMove(dy: number, eventTs: number): void {
  if (this.isOutOfBounds()) {
    dy *= 0.55; // 阻力系数，越界时拖动距离打折
  }
  this.offset += dy;
  this.addSample(this.offset, eventTs);
}
```

### 惯性越界回弹

惯性滚动超出边界后，用弹簧回弹：

```typescript
update(dt: number): void {
  // ... 惯性减速 ...

  if (this.isOutOfBounds()) {
    const target = this.clampToBounds(this.offset);
    const spring = Math.min(1, dt * 0.008);
    this.offset += (target - spring) * spring;

    if (Math.abs(this.offset - target) < 0.5) {
      this.offset = target;
      this.velocity = 0;
      this.isMomentumActive = false;
    }
  }
}
```

---

## 面板需实现的接口

供 InputHandler 调用的统一接口：

```typescript
interface ScrollablePanel {
  isPanelVisible(): boolean;
  handleScroll(dy: number, eventTs?: number): boolean;
  startMomentum(velocity: number, eventTs?: number): boolean;
  stopMomentum(): void;
}
```

---

## 踩坑记录

| 问题 | 现象 | 原因 | 修复 |
|------|------|------|------|
| 移动端无惯性 | 桌面 Chrome 正常，手机完全无惯性 | 使用 `performance.now()` 而非 `event.timeStamp`，移动端主线程延迟导致速度失准 | 全链路传递 `event.timeStamp` |
| 快速短甩无惯性 | 慢速长滑有惯性，3-5 次 move 的快甩没有 | 指数平滑使短甩速度只达实际 40-60%，低于阈值 | 改用采样窗口法 |
| 陈旧阈值过紧 | touchend 距 touchmove > 40ms 就归零 | 移动端最后 touchmove→touchend 间隔常 40-100ms | 阈值从 40ms 调至 100ms |
| 衰减太快 | 惯性"一闪而过" | `0.985^(dt/16)` 衰减约 440 帧停止 | 改为 `0.998^dt`（dt 为 ms） |
| handleUp 丢失事件 | 事件对象被丢弃 | `onPointerUp = this.handleUp` 直接引用无参箭头函数 | 改为 `(e) => this.handleUp(e)` |
