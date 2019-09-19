---
title: VUE实现前端视频录制和截图
date: 2019-09-17 15:15:18
categories: VUE
---

## 介绍

该功能是基于`vue-video-player`和`recordrtc`两个库来实现的。视频录制功能大多在后端实现，但是会出现不同步的情况，所以如果在直播或者视频播放时，想要录制一个简短的视频同时又要保证一致性的情况下，可以选择在前端进行视频的录制。
<!-- more -->

## 依赖

```bash
npm install vue-video-player recordrtc
```

## 引入

因为`vue-video-player`文档比较坑爹，如果按照文档的方式引入，会导致视频流解析不了。具体引入方式应该为：

```javascript
//组件内引入
import { videoPlayer } from 'vue-video-player';
import 'videojs-contrib-hls';
require('video.js/dist/video-js.css');
require('vue-video-player/src/custom-theme.css');

//全局引入
import VideoPlayer from 'vue-video-player';
import 'videojs-contrib-hls';
require('video.js/dist/video-js.css');
require('vue-video-player/src/custom-theme.css');
Vue.use(VideoPlayer);
```
然后在需要使用的组件内引入`recordrtc`

```javascript
import RecordRTC from 'recordrtc'
```

## 截图

主要思路：    
1. 获取当前播放的`video`标签；
2. 创建一个`canvas`；
3. 调用`canvas`的`drawImage`方法进行绘制；
4. 将`canvas`转换为图片或base64；

### 获取video标签

因为在vue项目中，可以利用`ref`属性获取到video标签。我使用的是这种方法，也有其他的方法可以获取到，自由发挥就好。

```javascript
//获取video元素
let video = this.$refs.videoPlayer;
let videoEl = video.player.tech({
    IWillNotUseThisInPlugins: true
}).el();
```

### 创建canvas

```javascript
//创建一个canvas
let canvasEl = document.createElement('canvas');
canvasEl.width = videoEl.videoWidth;
canvasEl.height = videoEl.videoHeight;
```

### 调用drawImage方法

具体关于`drawImage`方法的使用，可以参考[文档](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage "drwaImage文档")

```javascript
//截图
let ctx = canvasEl.getContext('2d');
ctx.drawImage(videoEl, 0, 0, 544, 960, 0, 0, 544, 960);
```

### 转换为base64

```javascript
let image = canvasEl.toDataURL('image/png');
```

## 视频录制

实现思路：    
- 开始录像    
    1. 获取video标签；
    2. 获取视频流；
    3. 将视频流注入到`recordrtc`；
    4. 调用录制事件；
- 停止录像       
    1. 获取到录制的`blob`对象；
    2. 生成可以直接播放的`url`；
    2. 将`blob`转换为视频文件；

### 开始录像

```javascript
//获取video元素
let video = this.$refs.videoPlayer;
let videoEl = video.player.tech({
    IWillNotUseThisInPlugins: true
}).el();

//获取视频流
let stream = videoEl.captureStream();

//将视频流注入到recordRTC
this.recorder = RecordRTC(stream, {
    type: 'video'
});

//开始录制
this.recorder.startRecording();
```

### 停止录像

```javascript
//停止录制
this.recorder.stopRecording(() => {

    //获取录制的blob对象
    let blob = this.recorder.getBlob();
    this.videoFile = this.recorder.getBlob();

    //将blob转换为可以供video播放的url
    let url = URL.createObjectURL(blob);

    //将blob对象转换为文件
    let fileName = this.videoFileName+".webm";
    let fileObject = new File([blob], fileName, {
        type: 'video/webm'
    });     
    
});
```

## 录制插件

如果觉得自己实现太麻烦可以直接下载我的包，具体如何使用可以去[这里](https://github.com/lp-liupan/vue-video-tape "vue-video-tape")