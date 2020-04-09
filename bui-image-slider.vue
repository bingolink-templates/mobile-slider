<template>
    <slider :style="sliderStyle" :interval="interval" :auto-play="autoplay" :infinite="infinite"
            @change="_change">
        <div :key="index" v-for="(item,index) in items">
            <bui-image @click="_click($event,index)"
                       :width="imgWidth"
                       :resize="imgResize"
                       :height="imgHeight"
                       :placeholder="placeholder"
                       :src="item.url">
            </bui-image>
            <div  class='mode'></div>
            <text class='text' :style='authorStyle'>{{item.title}}</text>
        </div>
        <indicator class="indicator" :style="indicatorStyle"></indicator>
    </slider>
</template>

<style>
    .mode {
        position: absolute;
        bottom: 0;
        left: 0;
        width: 750px;
        height: 40wx;
        opacity: 0.7;
        background-image: linear-gradient(to bottom, #fff, #000);
    }
    .text {
        position: absolute;
        bottom: 0;
        left: 0;
        padding-left: 2wx;
        line-height: 40wx;
        color: #fff;
        lines: 1;
        overflow: hidden;
        text-overflow: ellipsis;
    }
    .indicator {
        width: 750px;
        height: 100px;
        position: absolute;
        bottom: -5wx;
        left: 0;
        item-color: #ffffff;
        item-selectedColor: #747474;
        item-size: 4wx;
        opacity: 0.8;
    }
</style>
<script>
    module.exports = {
        props: {
            sliderStyle: {
                type:Object,
                default: {
                    "width": "750px",
                    "height": "400px"
                }
            },
            indicatorStyle: {
                type:Object,
                default:function () {
                    return {};
                }
            },
            authorStyle: {
                type:Object,
                default:function () {
                    return {};
                }
            },
            items: {
                type:Array,
                default:function () {
                    return [];
                }
            },
            interval: {
                type:Number,
                default: 6000 //ms
            },
            autoplay: {
                type:Boolean,
                default: true
            },
            infinite: {
                type:Boolean,
                default: true
            },
            imgResize: {
                type:String,
                default: "stretch"
            },
            imgWidth: {
                type:[Number,String],
                default: 750
            },
            imgHeight: {
                type:[Number,String],
                default: 750
            },
            placeholder: {
                type:String,
                default: ""
            }
        },
        methods: {
            _change(e) {
                this.$emit("change", e);
            },
            _click(e, index) {
                this.$emit("itemClick", e, index);
            }
        }
    }
</script>
