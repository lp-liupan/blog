---
title: 封装VUE组件并发布到npm
date: 2019-09-18 16:44:24
categories: 开发小工具
---

## 前言

因为项目中遇到了部分需求在网上并没有找到比较好的解决方案，所以决定自己实现。实现过后本着为劳苦大众做贡献的想法决定将其开源，顺便也学习一下开源的流程。

>至于网上这么多教程，我还要单独写一篇。因为我遇到了很多坑，搞了好几天才完整的弄好，为了下次发布npm包方便，所以记录一下。

## 准备npm

1. 首先需要到npm官网去注册，[点这里去哦](https://www.npmjs.com/ "npm官网")。    
2. 然后在电脑上登录
```bash
npm login
```

## 项目准备
这里我们使用的是vue项目，而且需要发布的包的功能也不复杂，所以我们选择使用`webpack-simple`搭建一个简易的`vue`项目，使用`vue@cli`搭建的脚手架对我们来说太臃肿了。
```bash
vue init webpack-simple 项目名  

#然后会提示填写作者，项目名等信息，不想写就直接‘y’或者‘enter’

cd 项目文件

npm install
```
`npm`下载完毕后就可以直接运行项目了
```bash
npm run dev
```
此时项目的目录主要结构应该是如下这样的
```
|-src
|-|-assets
|-|-|-logo.png
|-|-App.vue
|-|-main.js
|-index.html
|-package.json
|-webpack.config.js
```

## 项目开发

可以先改造一下项目目录结构，下面是我改造后的，也可以根据自己的编码习惯改造，但是增加的两个`index.js`是比较关键的文件，不要漏掉了。
```
|-src
|-|-tape         //自定义文件夹
|-|-|-tape.vue   //自定义组件名
|-|-|-index.js   //注意这个文件
|-|-App.vue
|-|-main.js
|-|-index.js     //注意这个文件
|-index.html
|-package.json
|-webpack.config.js
```
{% asset_img 目录结构.png %}

***
注意：
在编辑需要发布到npm的组件时（这里就是指`tape.vue`组件），务必要记得不能忽略`name`属性，这个涉及到后面组件导出的问题。

## 发布

在实现了功能过后，我们可以利用`App.vue`来测试一下组件的功能
```html
<template>
    <div id="app">
       <vueVideoTape ></vueVideoTape>
    </div>
</template>
<script>

import vueVideoTape from './tape/tape'

export default  {
    data(){
        return{}
    },
    components:{
       vueVideoTape
	},
	methods:{
	
	}
}
</script>
```

>经过测试没有问题后，就可以开始着手发布前的准备了，这里也是最麻烦的地方，很多的小细节要注意。

首先要对组件的导出做处理，要实现其他人通过`npm install`下载后，可以全局引入，也可以组件内按需引入。也就是要实现下列这两个引用方式。
```javascript

//全局引入
import tape from 'vue-video-tape';
Vue.use(tape);

//组件内按需引入
import {VueVideoTape} from 'vue-video-tape';
```

与`tape.vue`同级`index.js`对组件导出做如下处理
```javascript
import VueVideoTape from './tape.vue';

//这里VueVideoTape.name就是组件中的name属性，所以正如前文提到到，name属性不可以忽略，必须要有
VueVideoTape.install = Vue => Vue.component(VueVideoTape.name, VueVideoTape);

export default VueVideoTape;
```

与`main.js`同级的`index.js`对组件导出做如下处理
```javascript
//如果有多个导出的组件，在这里要引入
import VueVideoTape from './tape/index.js';


//如果引入多个组件，在这里都要注册
const components = [
	VueVideoTape,
]

const install = function(Vue, opts = {}) {
  components.map(component => {
    Vue.component(component.name, component);
  })
}

if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue);
}


//多个组件时，在这里要全部导出，install不可以忽略，全局引入时，vue会自动调用install
export default {
  install,
  VueVideoTape,
}

//多个组件导出，这里依次导出，组件内按需引入时就是引入这里的组件
export {
	VueVideoTape
}
```

***

>文件的导出处理完后，还需要对webpack的配置进行修改。    

首先是对开发环境和生产环境的区分，生产环境就直接打包到`dist`文件夹。
```javascript
entry: NODE_ENV == 'development' ? './src/main.js' : './src/index.js',
  output: {
    path: path.resolve(__dirname, './dist'),//打包输出地址
    publicPath: '/dist/',
    filename: 'vue-video-tape.js',//打包后文件名
    library:'VueVideoTape',
	libraryTarget:'umd',
	umdNamedDefine: true
  },
```

>开发功能时候也会需要引入其他的`npm`包，但是我们打包发布的时候，要将这些引入的包排除，否则就会导致打包的文件会非常大，而且引入过后造成依赖重复，冗余非常严重。所以在生产环境的时候，我们要对打包的文件进行单独处理。
```javascript
//在webpck.config.js最后加上下面的代码
if (process.env.NODE_ENV === 'production') {
  module.exports.devtool = '#source-map'
  module.exports.plugins = (module.exports.plugins || []).concat([
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    }),
    new webpack.optimize.UglifyJsPlugin({
      sourceMap: false,//这里改为false
      compress: {
        warnings: false
      }
    }),
    new webpack.LoaderOptionsPlugin({
      minimize: true
	}),
	new webpack.optimize.UglifyJsPlugin({
		compress: {
		  warnings: false
		}
	  })
  ])

//这里是对组件内依赖的排除
  module.exports.externals = {
	recordrtc: {
        commonjs: 'recordrtc',
        commonjs2: 'recordrtc',
        amd: 'recordrtc',
        root: '_'
	},
	'vue-video-player': {
        commonjs: 'vue-video-player',
        commonjs2: 'vue-video-player',
        amd: 'vue-video-player',
        root: '_'
	},
  }
}
```

webpack配置修改完后，还需要对package.json进行部分修改，主要涉及到几个发布npm包需要用到的关键属性。

`name`属性决定了`npm install`时候的名字
```json
"name": "vue-video-tape",
```
```bash
npm install vue-video-tape
```

`version`属性是发布的版本，`每次发布`都必须要对其进行修改，否则发布失败

```json
"version": "1.1.0",
```

`private`属性是决定是否开源，如果为`true`则发布不了

```json
"private": false,
```

`main`属性决定了当`import xxx from 'xxxxx'`时，引入哪个文件

```json
"main": "dist/vue-video-tape.js",
```

每次发布时我们都需要对项目进行打包，但是经常会忘记打包直接发布，导致发布的新版本是上一个版本，所以我们可以引入一个事件，在每次发布的前自动进行打包操作。
```json
"scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --open --hot",
    "build": "cross-env NODE_ENV=production webpack --progress --hide-modules",
    "prepublishOnly": "npm run build"
  },
```

`files`属性指定当`npm install`时下载的文件，如下配置后，`npm install vue-video-tape`下载的包就只有dist文件夹和package.json文件，可以减少包的体积。
```json
"files": [
    "dist",
    "package.json"
  ],
```

***

>最后的最后，当一切准备就绪了，就可以执行发布命令了
```bash
npm publish
```
注意：可能会报错你的项目名字已经有了，这时候你需要去修改`package.json`文件中的`name`属性。     

发布成功后，可以直接到npm官网上搜索你的包。


## 下载使用

```bash
npm install 包名
```

全局引入
```javascript
import xxx from 'vue-video-tape';
Vue.use(xxx);
```

组件内引入
```javascript
import VueVideoTape from 'vue-video-tape';

components:{VueVideoTape},
```