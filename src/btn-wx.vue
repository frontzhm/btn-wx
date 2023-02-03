<script>
/**
 * import btnWx from './components/btnWx';
 * <btnWx v-if="configWx" class="btn" :configCallAppNotInWx="configCallAppNotInWx" :configWx="configWx" :configWxTag="configWxTag" >打开APP</btnWx >
 * .btn {..btn本身的样式}
 */
import wx from 'weixin-js-sdk';
import CallApp from 'callapp-lib';
const isInWx = window.navigator.userAgent
  .toLowerCase()
  .includes('micromessenger');
export default /* #__PURE__ */ {
  name: 'BtnWx', // vue component name
  props: {
    // 属性 appId, timestamp, nonceStr, signature ,debug
    configWx: {
      type: Object,
    },
    // 属性 appWxId extinfo
    configWxTag: {
      type: Object,
    },
    // scheme package appstore yingyongbao downloadPage fallback timeout
    configCallAppNotInWx: Object,
    clickBtnNotInWx: Function,
  },

  data() {
    return {
      isInWx,
    };
  },
  created() {
    this.initCallApp();
    // 这里防止不是异步的
    this.isInWx && this.configWx &&  this.setWxConfig();
  },
  watch: {
    // 这里尤其注意，configWx可能是异步的，所以需要之后的值
    configWx() {
      this.isInWx && this.setWxConfig();
    },
  },
  methods: {
    // 微信按钮成功打开APP
    openAppSuccess() {
      if (this.configWxTag.launch) {
        this.configWxTag.launch();
        return;
      }
      console.log('恭喜！成功跳转到APP', this._props);
    },
    // 微信按钮没有成功打开APP,就跳转到下载页
    openAppFail(e) {
      if (this.configWxTag.error) {
        this.configWxTag.error(e);
        return;
      }
      console.log(
        `跳转APP失败，你可能有点难过，别急，有对策!\n
      先把deBug设置为true，如果config都失败了，去调config。\n
      config过了，如果看到isTrusted:false的话，APP下载没？\n
      APP确定下载的话，你是不是直接点击链接了？\n
      微信重场景，或者你收藏下链接再打开，或者点击分享卡片，或者变成二维码扫，应该就可以啦!\n
      参考https://developers.weixin.qq.com/community/develop/doc/00022492a18c68ff972d06bcc51800?source=indexmixflow
      `,
        e,
        e.detail,
        this._props
      );
      if (this.configCallAppNotInWx.downloadPage) {
        location.href = this.configCallAppNotInWx.downloadPage;
      }
    },
    async setWxConfig() {
      wx.config({
        debug: false,
        jsApiList: [],
        openTagList: ['wx-open-launch-app'],
        ...this.configWx,
      });
    },
    initCallApp() {
      const {
        scheme,
        package: packageRename,
        appstore,
        yingyongbao,
        fallback,
        timeout,
      } = this.configCallAppNotInWx;
      console.log(this.configCallAppNotInWx);
      this.callLib = new CallApp({
        scheme: { protocol: scheme },
        intent: { package: packageRename, scheme },
        appstore,
        yingyongbao,
        timeout: timeout || 4500,
        fallback,
      });
      console.log(this.callLib);
    },
    // 在浏览器的话，打开app
    callApp() {
      if (this.clickBtnNotInWx) {
        this.clickBtnNotInWx();
        return;
      }
      this.callLib.open({ path: this.configWxTag.extinfo });
    },
  },
};
</script>

<template>
  <div class="btn-wx" v-on="$listeners">
    <div style="position: relative">
      <div @click="callApp"><slot /></div>
      <!-- 微信环境里，微信标签按钮定位在普通按钮上 -->
      <wx-open-launch-app
        v-if="isInWx"
        :appId="configWxTag.appWxId"
        :extinfo="configWxTag.extinfo"
        @launch="openAppSuccess"
        @error="openAppFail"
        style="
          position: absolute;
          top: 0;
          left: 0;
          bottom: 0;
          right: 0;
          z-index: 10;
        "
      >
        <script type="text/wxtag-template">
          <!-- 这里的按钮就是浮在，普通按钮上面，和普通按钮同宽同高 -->
          <div style="position: absolute; top: 0; left: 0;bottom:0;right:0; z-index: 10;"></div>
        </script>
      </wx-open-launch-app>
    </div>
  </div>
</template>
