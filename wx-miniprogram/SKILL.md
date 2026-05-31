---
name: wx-miniprogram
description: 微信小程序开发专家指南，覆盖框架原理、WXML/WXSS/JS 规范、生命周期、Canvas 2D、网络、分包，以及实战坑记录。当用户提到微信小程序开发、wxml、wxss、Page()、app.json、小程序 canvas、scroll-view、wx.request、分包、组件开发、小程序部署上线等场景时必须使用此 skill。也适用于小程序报错，例如"WXSS 编译错误"、"canvas 节点获取失败"、"statusCode 不对"、"域名不合法"、"小程序包体积超限"等。即使用户没有明确说"小程序"，只要涉及 app.json 配置、Page({}) 注册、WXML 模板语法，也应触发此 skill。官方文档：https://developers.weixin.qq.com/miniprogram/dev/framework/
user_invocable: true
---

# 微信小程序开发专家指南

官方文档：https://developers.weixin.qq.com/miniprogram/dev/framework/

需要查阅具体 API 时，直接用 WebFetch 抓取对应文档页面。本 skill 提供核心原则和实战积累的坑记录。

---

## 核心架构原则

小程序采用**逻辑层与渲染层分离**架构：

- **逻辑层**（JS）：运行在独立的 JS 引擎（iOS: JavaScriptCore，Android: V8，工具: NWJS）
- **渲染层**（WXML/WXSS）：独立线程处理 UI，不阻塞逻辑层
- **通信**：两层通过框架桥接，数据绑定 `{{var}}` 驱动 UI 更新

**重要限制（与 Web 最大区别）**：
- 无 DOM/BOM API（没有 `document`、`window`、`querySelector`）
- `eval()` 和 `new Function()` 被禁用（安全限制，唯一例外：`new Function('return this')`）
- 不能使用依赖 DOM 的库（jQuery、Zepto 等）
- 只能通过 `setData()` 更新 UI，直接修改 `this.data` 无效

## JS 运行时注意事项

- **Promise 在 iOS 15 及以下行为异常**：Promise 微任务按宏任务顺序执行，链式调用顺序可能错乱；升级基础库或避免依赖精确微任务顺序
- **`Proxy` 在旧客户端不可靠**：避免使用，或做版本检测
- **没有 `setTimeout`/`setInterval` 的精确性保证**：小程序后台状态下定时器可能被暂停

---

## 实战坑记录

遇到新坑在此追加，格式：**现象 → 根因 → 修复**。

---

### 坑 1：WXSS 不支持 `*` 通配选择器

**现象**：`app.wxss` 编译报错 `unexpected token '*'`

**根因**：WXSS 不支持 `*`、`*::before`、`*::after`

**修复**：删除，按需在具体组件上设置

```css
/* ❌ */ *, *::before, *::after { box-sizing: border-box; }
/* ✅ 删掉，或写具体类名 */
```