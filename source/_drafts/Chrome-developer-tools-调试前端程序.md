---
title: Chrome developer tools 调试前端程序
comments: true
date: 2019-07-16 06:52:37
tags: chrome
categories: 浏览器
---

[chrome-devtools 官方教程](https://developers.google.com/web/tools/chrome-devtools/)


## 打开Chrome 开发者工具
- 在Chrome菜单中选择 更多工具 > 开发者工具
- 在页面元素上右键点击，选择 “检查”
- 使用 快捷键 <code>⌃</code>+<code>⇧</code>+<code>I</code> (Windows) 或 <code>⌘</code>+<code>⌥</code>+<code>I</code> (Mac)

## 面板

### 1. 设备模式
> 使用设备模式构建完全响应式，移动优先的网络体验。
> **功能要点**：
> 1. 测试完全响应式设计
> 2. 测试不同网络条件的性能
> 3. 模拟传感器

- Warning: Device Mode 可以让您了解您的网站在移动设备上的大致显示效果，但要获得全面的了解，则应始终在真实设备上测试您的网站。DevTools **无法模拟移动设备的性能特性。**
- <code>⌘</code>+<code>⇧</code>+<code>M</code> (Mac) 或 <code>⌃</code>+<code>⇧</code>+<code>M</code>（Windows、Linux）来切换 Device Mode

- [测试自适应和设备特定的视口](https://developers.google.com/web/tools/chrome-devtools/device-mode/emulate-mobile-viewports)
- [在不断变化的网络条件下优化性能](https://developers.google.com/web/tools/chrome-devtools/network-performance/network-conditions)
- [模拟传感器：地理定位与加速度计](https://developers.google.com/web/tools/chrome-devtools/device-mode/device-input-and-sensors)

---

### 2. 元素面板
> 使用元素面板可以自由的操作DOM和CSS来迭代布局和设计页面。
> **功能要点**：
> 1. 检查页面的DOM结构与样式以及盒模型
> 2. 实时编辑DOM与CSS,并查看效果，可快速迭代出页面设计
> 3. 重新加载页面时，实时编辑都会丢失，可尝试[设置持久化](https://developers.google.com/web/tools/setup/setup-workflow)来满足需求

 - [使用 DevTools 的工作区设置持久化](https://developers.google.com/web/tools/setup/setup-workflow)

### 3. 控制台面板
> 在开发期间，可以使用控制台面板记录诊断信息，或者使用它作为 shell在页面上与JavaScript交互。
### 4. 源代码面板
> 在源代码面板中设置断点来调试 JavaScript ，或者通过Workspaces（工作区）连接本地文件来使用开发者工具的实时编辑器。
### 5. 网络面板
> 使用网络面板了解请求和下载的资源文件并优化网页加载性能。
### 6. 性能面板
> 使用时间轴面板可以通过记录和查看网站生命周期内发生的各种事件来提高页面的运行时性能。
### 7. 内存面板
> 如果需要比时间轴面板提供的更多信息，可以使用“配置”面板，例如跟踪内存泄漏。 Use the Profiles panel if you need more information than the Timeline provide, for instance to track down memory leaks.
### 8. 应用面板
> 使用资源面板检查加载的所有资源，包括IndexedDB与Web SQL数据库，本地和会话存储，cookie，应用程序缓存，图像，字体和样式表。
### 9. 安全面板
> 使用安全面板调试混合内容问题，证书问题等等。
