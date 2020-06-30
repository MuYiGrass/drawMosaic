<template>
  <div id="app">
    <mosaic :imgUrl="imageUrl"
            ref="mosaic"
            :tileData="tileData"
            :maxWidth="maxWidth"
            :minWidth="minWidth"
            @submitImage="outputImage">
    </mosaic>
    <img :src="outImageUrl"
         class="outImage">
    <button @click="drawAllTiles">填充全部</button>
    <button @click="eraseAllTiles">清除全部</button>
    <button @click="submitCanvas">提交</button>
    <button @click="addMosaic">增加马赛克大小</button>
    <button @click="addBrush">增加画笔大小</button>
  </div>
</template>

<script>
import mosaic from './components/mosaic.vue'

export default {
  name: 'App',
  data () {
    return {
      // imageUrl: require('./assets/test.jpg'),
      imageUrl: 'https://u.thsi.cn/imgsrc/sns/3bc9c557b805b1808e84107e25818d32_675_698.jpg',
      outImageUrl: '',
      tileData: {
        tileWidth: 5,
        tileHeight: 5,
        brushSize: 3
      },
      maxWidth: 600,
      minWidth: 300,
    }
  },
  components: {
    mosaic,
  },
  methods: {
    outputImage (e) {
      this.outImageUrl = e
    },
    drawAllTiles () {
      this.$refs.mosaic.drawAllTiles()
    },
    eraseAllTiles () {
      this.$refs.mosaic.eraseAllTiles()
    },
    submitCanvas () {
      this.$refs.mosaic.submitCanvas()
    },
    addMosaic () {
      this.tileData.tileWidth++;
      this.tileData.tileHeight++;
    },
    addBrush () {
      this.tileData.brushSize++
    }
  }
}
</script>

<style lang="less" scoped>
#app {
  .mask {
    width: 80%;
    height: 50%;
    background-color: rgba(0, 0, 0, 0.2);
    margin-left: 50px;
    overflow: scroll;
    outline: solid red 1px;
  }
  .outImage {
    position: absolute;
    top: 0;
  }
}
</style>
