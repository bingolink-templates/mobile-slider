<template>
    <div style="flex:1">
        <slider></slider>
    </div>
</template>

<!--引入bui-weex样式文件-->
<style lang="scss" src="bui-weex/src/css/buiweex.scss"></style>

<script>
    import slider from "./slider.vue";
    module.exports = {
        data () {
            return {}
        },
        components: {
          slider
        }
    }
</script>