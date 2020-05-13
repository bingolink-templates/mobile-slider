<template>
  <div style="flex:1;background-color:#fff">
    <imageSlider></imageSlider>
  </div>
</template>

<!--引入bui-weex样式文件-->
<style lang="scss" src="bui-weex/src/css/buiweex.scss"></style>

<script>
  import imageSlider from "./slider.vue";
  module.exports = {
    data() {
      return {}
    },
    components: {
      imageSlider
    }
  }
</script>