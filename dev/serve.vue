<script>
import Vue from 'vue';
import BtnWx from '@/btn-wx.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    BtnWx
  }
});
</script>

<template>
  <div id="app">
    <btn-wx />
  </div>
</template>
