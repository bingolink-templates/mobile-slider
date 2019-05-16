<template>
  <div class="news">
      <!-- 新闻轮播图 -->
      <div class="slider-common">
        <image-slider style="height:400px;" auto="true" pos="1" :items="sliderItems" @onItemClick="sliderEvent"
          radius='10'></image-slider>
      </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        sliderItems: [{
            title: '个性需求接受',
            url: 'https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=595509099,562291008&fm=173&app=49&f=JPEG?w=218&h=146&s=808401F358338F9672087E6C0300E054'
          },
          {
            title: '2',
            url: 'https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=595509099,562291008&fm=173&app=49&f=JPEG?w=218&h=146&s=808401F358338F9672087E6C0300E054'
          }, {
            title: '3',
            url: 'https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=595509099,562291008&fm=173&app=49&f=JPEG?w=218&h=146&s=808401F358338F9672087E6C0300E054'
          }
        ],
      }
    },
    methods: {
      // 轮播图
      sliderEvent() {
        this.$alert('轮播')
      }
    }
  }
</script>

<style lang="css" src="../css/common.css"></style>
<style>
  .news {
    flex: 1;
    background-color: #f2f2f2;
  }

  .slider-common {
    position: relative;
    background-color: #fff;
    margin-bottom: 20px;
    border-radius: 10px;
    height: 420px;
  }
</style>