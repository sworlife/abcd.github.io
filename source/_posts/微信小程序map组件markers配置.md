---
title: 微信小程序map组件markers 的 iconPath 临时路径的应用
comments: true
date: 2019-10-23 09:19:23
tags: 微信小程序
categories: 前端技术
---
> 代码的世界就是这样变换莫测，今天刚写好的 blog，马上就过期了
[小程序基础库更新日志](https://developers.weixin.qq.com/miniprogram/dev/framework/release/) v2.9.0 (2019-10-09) 第17条：U 更新 组件 map 增加 label 点击事件 反馈详情

## 需求
在一个地图上需要根据后端接口提供的经纬度进行标注，标注内容是动态的，包括：城市，价格等。点击 markers 需要触发事件，拉起一个弹窗

## 需求分析
在这个需求分析中，我们只关注 markers 的配置，以及 markers 事件触发

根据文档，很容易找到 markers 的相关配置，比如经纬度（longitude， latitude），图标（iconPath，rotate，alpha， width， height，anchor），标签（label）

## 实现思路
通过 iconPath 来配置 marker 图标，城市、价格等动态内容通过 label 来配置

## 遇到的问题
label 的层级比 iconPath 高，markers 点击事件触发确只能通过点击 iconPath 触发，就导致 label 遮挡 自身及其他 marker 的 iconPath，使得 marker 点击事件无法触发。尤其数据量大，或者地图缩小，使得数据集中显示。按用户的角度来看，我点击一个 marker，因为数据集中，响应的数据可与与我的预期不一致，但不能不响应。

## 解决方案
既然 label 导致无法触发事件，那就只能放弃，剩下只剩一条路：把动态数据以及图标转换为**临时路径**，全部通过 iconPath 来实现。

临时路径可以通过 `wx.canvasToTempFilePath` 来获取, 这就需要 `canvas` 组件，由于微信小程序无法通过 js 创建节点，所以需要在 `.wxml` 中事先声明好 `canvas` 组件。因此，大概流程是这样：

首先创建模版
```html5
// .wxml
<map></map>
<canvas></canvas>
```
然后获取数据
```js
ajax.get().then(markers => {
  // 将数据画在 canvas 上
  drawImage(markers);
  // 将 canvas 转换为临时路径
  getTempFilePath(markers);
})
```

但最好发现，因为 `ctx.draw`，`wx.canvasToTempFilePath` 都是异步操作，因此只通过一个 `canvas` 节点如果要完成大量数据（先绘图，在转换临时路径）操作，就需要依次排队处理，这不但代码难以实现，同时也导致处理时间延长了。

因此，我们需要针对每一个数据，都分配一个 `canvas` 组件，然后并行处理（先绘图，在转换临时路径），利用 `Promise.all` 来等待所以异步操作完成时，在返回结果，更新 markers，实现需求。

**注意事项：**
- 动态生成 canvas 组件渲染完成后，才能进行绘图、转换为临时路径
- `ctx.draw`，`wx.canvasToTempFilePath` 都是回调式的 API，因此需要转换为 Promise 式的 API

## 主要代码

```html
<!-- .wxml -->
<map 
  markers="{{markers}}" 
>
</map>
<block wx:for="{{canvasList}}" wx:key="{{item.id}}">
  <canvas canvas-id="markerIcon-{{index}}"><canvas>
</block>
```
```js
_getMarkers() {
  service.getMarkers().then(markers => {
    const canvasList = markers.map(({id}) => ({id}));
    // 先利用 canvasList 渲染 canvas，后 画图获取 iconPath
    this.setData({ canvasList }, () => this._drawMarkersIcon(markers));
  }, err => {
    console.error(err);
  });
},


_drawMarkersIcon(markers) {
  return Promise.all(
    markers.map((_item, index) => {
      const canvasId = `markerIcon-${index}`;
      const ctx = wx.createCanvasContext(canvasId);
      // 绘图
      drawImage(ctx)
      // ctx.draw,wx.canvasToTempFilePath 转换为 Promise, 最终将返回 marker 配置
      return new Promise((resolve) => {
        ctx.draw(false, () => {
          resolve(
            new Promise((resolve, reject) => {
              wx.canvasToTempFilePath({ 
                canvasId,
                success: resolve,
                failed: reject
              })
            }).then(res => {
              _item.iconPath = res.tempFilePath;
              return _item;
            }, err => {
              console.error(err);
            })
          );
        });
      });
    })
  ).then(markers => {
    this.setData({ markers });
  });
}
```

## 总结
- 微信小程序 map 组件的 markers 事件触发只能通过 iconPath 配置的图标触发。其他诸如 label 类的元素还会起到遮挡作用。因此，label 这个属性在需要事件触发的情况下，基本是废掉，不应该考虑使用。
- 利用 canvas 来定制数据图片，并用于界面展示是可行的。这个能力应该能应用于非小程序（web、h5）,且实现更加优雅（js 创建/删除元素，与界面解藕）
