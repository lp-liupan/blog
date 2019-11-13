---
title: VUE中使用flvjs踩坑
date: 2019-11-13 16:13:48
categories: VUE
---

# 在vue项目中使用flvjs

安装
```bash
npm install flv.js
```

组件内引入
```js
import flvjs from 'flv.js'
```

使用：

1. 创建一个id为`videoElement`的`video`标签；
2. 在生命周期`mounted`中使用flvjs，具体使用范例如下；
	```html
	<template>
		<div id="app">
			<video id="videoElement" controls autoplay muted width="600" height="450"></video>
		</div>
	</template>

	<script>

	import flvjs from 'flv.js'
	export default {
	name: 'app',
	data(){
		return{}
	},
	mounted(){
		if (flvjs.isSupported()) {
			let videoElement = document.getElementById('videoElement');
			let flvPlayer = flvjs.createPlayer({
				type: 'flv',
				isLive: true,
				hasAudio: false,
				url: 'http://www.xxxxxxx.com:18080/11/22.flv'
			});
			console.log(flvPlayer,'flv对象')
			flvPlayer.attachMediaElement(videoElement);
			flvPlayer.load();
			flvPlayer.play();
		}

	},
	methods:{
		

	}
	}
	</script>

	```

# 播放失败原因

## 协议不支持

在一开始我以为flvjs可以播放所有flv格式的视频流，但是经过测试和查看文档发现，这个包仅支持`HTTPFLV`协议的流，如果使用`RTMP`协议的流则依然需要使用flash插件。

>支持：http://www.xxxxxxx.com:18080/11/22.flv

>不支持：rtmp://www.xxxxx.com/api/6538-1.1567494734966.flv

## 音频流没有

现在在物联网行业中有非常多的摄像头需要接入到web中，而摄像头往往是没有音频的。这时候如果你在界面中看到视频一直在加载，就像下图这样：

{% asset_img 视频加载.png %}  

这时候就要考虑是不是直播流中没有包含音频流，如果没有就要设置如下：
```js
let flvPlayer = flvjs.createPlayer({
				type: 'flv',
				isLive: true,
				hasAudio: false, //设置这个参数为false
				url: 'http://www.xxxxxxx.com:18080/11/22.flv'
			});
```

## 自动播放失败

{% asset_img 静音报错.png %} 

视频流的播放往往我们都选择自动开始播放，但是自动播放被谷歌等浏览器禁用了，需要单独进行设置。这对于用户体验非常的不好，所以经过一系列的百度谷歌，最后知道了实际上浏览器禁用的是音频自动播放，只要我们在`video`标签上设置`muted`属性即可。
```html
<video id="videoElement" controls autoplay muted width="600" height="450"></video>
```
