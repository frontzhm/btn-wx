---
title: 微信中打开自家APP - 现成按钮组件来了！
tags: vue
categories: vue
theme: vue-pro
highlight:
---

本组件主要是一个 vue 按钮组件，此按钮可在微信浏览器，直接唤起自家的 APP，背后借助的是[微信开放标签](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_Open_Tag.html)

注意！不能复制链接到对话框，然后点击哈！  
注意！不能复制链接到对话框，然后点击哈！  
注意！不能复制链接到对话框，然后点击哈！  

微信重场景，或扫码，或收藏里，或分享出来的卡片等等吧，反正不能直接点击赤裸裸的链接。如果配置都正常，但显示`{isTrusted:false}`的话，可能就是这个问题（天知道，就这，我排查了一天）。

## 演示效果

![c4.mp4](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/c4.mp4)

## 安装

```shell
# npm i btn-wx
yarn add btn-wx
```

## 简单使用

```vue
<template>
<btnWx class="btn"
  :configWx="configWx"
  :configWxTag="configWxTag"
  :configCallAppNotInWx="configCallAppNotInWx">
  打开自家APP
</btnWx >

</template>
<script>
import btnWx from 'btn-wx';

const configWx = {
  debug: false, // 默认的debug模式是关闭的，需要的话，设置为true
  appId: "wx69fd8", // appId、timestamp、nonceStr、signature一般从后端接口拿到
  timestamp: "1668766376",
  nonceStr: "f0a80f566a347bb",
  signature: "a3e217abf1900ecc0af16122878d6",
}

const configWxTag = {
  appWxId: "wxa123456", // app端和微信申请的
  extinfo:"talent://open/view/detail?code=2&id=1", // 跳转到app的路径，首页的话 talent://
};

const configCallAppNotInWx = {
  scheme: "talent",
  package: "cn.x",
  appstore: "https://apps.apple.com/cn/app/id736", // ios的下载地址
  yingyongbao: "", // 应用宝的下载地址
  downloadPage: "https://huahua.com/app/download", // h5的下载地址
  fallback: "http://vip-talent.firqr.com/talent" // 安卓app的下载地址
};


export default {
  data(){
    configWx,
    configWxTag,
    configCallAppNotInWx,
  }
}
<script>
<style scoped>
/* btn 样式自定义设置，不影响*/
.btn{color:#f69;}
</style>
```

## PROPS

### configWx

需要微信功能的基础配置

```js
const configWx = {
  appId: "wx69fd8",
  timestamp: "1668766376",
  nonceStr: "f0a80f566a347bb",
  signature: "a3e217abf1900ecc0af16122878d6",
  // 上面是必须的

  // 以下是默认的，设置的话就覆盖
  debug: false,
  jsApiList: [],
  openTagList: ["wx-open-launch-app"],
};
```

### configWxTag

需要微信开放标签的配置

```js
const configWxTag = {
  appWxId: "wxa123456", // app端和微信申请的
  extinfo: "talent://open/view/detail?code=2&id=1", // 跳转到app的路径，首页的话 talent://
  // 上面是必备的

  // 下面不是必须的，传就覆盖默认
  launch: () => {
    console.log("成功在微信里跳到APP了");
  },
  error: (e) => {
    console.log(
      "失败在微信里跳到APP了"
    ); /* 这边默认如果有下载地址，失败会默认跳到下载地址*/
  },
};
```

### configCallAppNotInWx

在非微信浏览器里，如需跳转 APP，那么需要的参数

```js
const configCallAppNotInWx = {
  scheme: "talent",
  package: "cn.x",
  appstore: "https://apps.apple.com/cn/app/id736", // ios的下载地址
  yingyongbao: "", // 应用宝的下载地址
  downloadPage: "https://huahua.com/app/download", // h5的下载地址
  fallback: "http://vip-talent.firqr.com/talent", // 安卓app的下载地址
};
```

### clickBtnNotInWx

在非微信浏览器，点击按钮的函数，默认会用配置`configCallAppNotInWx`唤起 APP

### 事件

直接继承父组件的事件。

## 组件开发的逻辑 - 可以参照此逻辑自己写组件

### 前提一般步骤

1. 绑定域名
   登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS 接口安全域名”。
1. 自家 app 端需要配合，接入相关 SDK，因为跳转的时候需要 app 和微信申请的`appId`，以及 app 端需要[按照微信的方式解析`extInfo`](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/APP_GET_EXTINF.html)

1. 引入 JS 文件 `import wx from 'weixin-js-sdk';`

1. 通过 config 接口注入权限验证配置并申请所需开放标签

```js
wx.config({ debug: true, openTagList: ["wx-open-launch-app"] }); // 需要使用的开放标签列表，其他配置跟别的微信接口差不多，不赘述 ;debug模式如果是新手的话，最好打开，这样避开配置错误的雷
```

### vue 中使用的话，需要加忽略标签配置

1. vue 文件中使用的话，需要再`main.js`那边加标签忽略`Vue.config.ignoredElements = ['wx-open-launch-app'];`

### 按钮大致逻辑

- 唤起 app 的按钮，必须使用微信的开放标签，这里**需要`APP`端和微信申请`APPID`**，作为标签参数，同时需要`extInfo`，是跳转所需额外信息，如果是首页的话，`extInfo`可以直接是`xx://`
- 如果页面有可能在浏览器打开，那么需要兼容浏览器的情况，用普通方式打开 APP
- 为了方便，一般写一个盒子，浏览器的按钮先写好，然后微信的按钮定位在其上就好，如果是微信，点的就是微信按钮，如果不是点的就是浏览器按钮
- 浏览器跳转打开 APP 的话，直接用下`call-lib`库，封装了细节，使用便捷

## 开发的时候注意

- debug 模式尽量开启，防止出现配置错误
- 网页地址域名一定在微信开发者那边配置了
- 本地开发，务必使用代理到**许可的域名**，手机上也开启代理调试
- 确定上面没问题，如果只跳到首页，没跳到路径，可能 APP 没解析成功，找 APP 联调
- 出现`{isTrusted:false}`一定看下[进入场景](https://developers.weixin.qq.com/community/develop/doc/00022492a18c68ff972d06bcc51800?source=indexmixflow)

## 懒得搜代理怎么配置的话

这里简单贴下步骤：

- charles 里加上代理路径
  ![charles_wx](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/charles_wx.png)
- 手机端和电脑在同一个无线网，设置代理，看下电脑的 ip 地址`ifconfig en0`
  ![c1](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/c1.png)
- 如果是首次的话，做完上面之后
  - 注意电脑端的 charles 会弹框让你同意，你必须点`同意`
  - 手机在默认浏览器输入地址`chls.pro/ssl`，一般会触发下载
  - 下载完先安装好
  - 再去信任它（ios 的话，设置 - 通用 - 描述文件与设备管理）
    ![c2](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/c2.png)
- 此时就可以在手机的微信里打开`认可的域名`，然后可以调试了。
- 跳转功能的麻烦点在于，涉及的部门多，多多沟通

## 演示效果

![c4.mp4](https://blog-huahua.oss-cn-beijing.aliyuncs.com/blog/code/c4.mp4)
