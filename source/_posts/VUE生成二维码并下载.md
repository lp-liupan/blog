---
layout: post
title: VUE生成二维码并下载
categories: VUE
---
# 开始前了解
>在很多项目中二维码是由后端生成，但其实也可以在前端生成二维码并且直接下载。在这里我们就要使用一个vue的插件“qrcodejs2”，这个插件可以比较方便快捷的生成二维码。具体信息可以去官网了解。   

# 引入插件
>在命令行执行 npm install qrcodejs2     

>在vue文件中 import QRcode from 'qrcodejs2'     

# 插件的使用

## 挂载点，我们需要把要生成的二维码挂载都哪里
>首先要在模版中添加一个id=qrcode的div标签    

 ```html
<template>
    <div id="qrcode">

    </div>
</template>
```

## 生成函数
>二维码是通过canvas标签绘制出来的，所以我们需要一个函数去生成。函数放在了methods里面。
```javascript  
qrcode(){
    document.getElementById("qrcode").innerHTML = ""; //清除上次二维码的数据
    let qrcode = new QRcode('qrcode',{
        width:100,
        height:100,
        text:"你是不是在做梦哦",

    })
},
```

## 执行函数
>这里需要值得注意的是，因为二维码是通过canvas画出来然后挂载到div上的，所以我们在什么时候画就比较关键了。根据vue的生命周期，我们要将qrcode函数放到mounted函数中执行，如果在放在这个函数之前的生命周期函数中执行，就导致错误，因为id=“qrcode”的div还没有生成。

# 下载
>我这里的下载思路是根据广大的互联网学习来的，因为要下载所以我们得这个canvas转换为图片，但是仅仅转换为图片用户也只能通过右键进行下载，用户体验不好。这时候我们往往希望有个下载按钮可以点击下载。     

>主要的思路就是：  
1. 获取到canvas标签；
2. 生成一个a标签；
3. 把canvas转化为图片然后放到文件夹中，再把图片信息挂载到a标签的属性上；
4. 给图片一个下载名，然后触发a标签的click事件；      

```javascript
download(){
    let myCanvas = document.getElementById('qrcode').getElementsByTagName('canvas');
    let a = document.createElement("a")
    a.href = myCanvas[0].toDataURL('image/png').replace('image/png', 'image/octet-stream')
    a.download = "文件名";
    a.click();
},
```  