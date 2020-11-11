<template>
  <div class="drawMosaic">
    <!-- 画布 -->
    <div class="canvas-container-mosaic">
      <!-- 刷子 -->
      <div class="mosaic-brush"
           :style="brushStyle"
           v-show="showBrush">
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'mosaic',
  props: {
    // 图片的url
    imgUrl: String,
    // 每个马赛克长，宽，刷子大小
    tileData: {
      type: Object,
      default: () => {
        return {
          tileWidth: 15,
          tileHeight: 15,
          brushSize: 5
        }
      },
    },
    minWidth: Number,
    maxWidth: Number,
  },
  components: {},
  data () {
    return {
      mosaic: '',
      tileRowSize: 0,       // 图片马赛克总行数
      tileColumnSize: 0,    // 图片马赛克总列数
      tiles: [],            // 图片每个马赛克信息
      context: null,        // canvas渲染上下文
      canvas: null,
      initImageData: [],    // 图片原始数据
      mouseData: null,      // 鼠标信息（位置）
      brushStyle: {},
      showBrush: false,
      timer: null,
    }
  },
  mounted () {
    this.initImage(this.imgUrl)
    this.addMouseEvent()
  },
  methods: {
    //初始化图片
    initImage (url) {
      let img = new Image();
      img.src = url;
      img.crossOrigin = "anonymous";
      img.onload = (e) => {
        e.target.style.maxWidth = `${this.maxWidth}px`
        e.target.style.minWidth = `${this.minWidth}px`
        this.initCanvas(e.target)
      };
    },
    //初始化canvas
    initCanvas (img) {
      this.canvas = document.createElement('canvas');
      this.canvas.classList.add("canvas-mosaic");
      this.context = this.canvas.getContext('2d');
      this.canvas.width = this.setImageSize(img).width;
      this.canvas.height = this.setImageSize(img).height;
      this.context.drawImage(img, 0, 0, this.canvas.width, this.canvas.height);
      this.initImageData = this.context.getImageData(0, 0, this.canvas.width, this.canvas.height).data;
      document.querySelector('.canvas-container-mosaic').appendChild(this.canvas);
      this.initMosaic(this.context);
    },
    //设置图片最大宽，高度
    setImageSize (img) {
      let temImg = {
        width: img.width,
        height: img.height,
      };
      if (this.minWidth && img.width < this.minWidth) {
        temImg.width = this.minWidth;
        temImg.height = temImg.width * (img.height / img.width);
      }
      if (this.maxWidth && img.width > this.maxWidth) {
        temImg.width = this.maxWidth;
        temImg.height = temImg.width * (img.height / img.width);
      }
      img.width = temImg.width;
      img.height = temImg.height;
      return temImg;
    },
    //给马赛克容器绑定鼠标事件（包括canvas，brush）
    addMouseEvent () {
      let target = document.querySelector('.canvas-container-mosaic');
      target.addEventListener("mousedown", (e) => {
        target.addEventListener('mousemove', this.handleMouseMove)
      });
      document.addEventListener('mouseup', (e) => {
        this.showBrush = false;
        target.removeEventListener('mousemove', this.handleMouseMove);
      });
    },
    //鼠标移动
    handleMouseMove (target) {
      let container = document.querySelector('.canvas-container-mosaic');
      //相对画布位置
      let x = target.pageX - container.getBoundingClientRect().left;
      let y = target.pageY - container.getBoundingClientRect().top;
      this.setBrushPosition(true, target, target);
      // 擦除马赛克
      if (target.shiftKey) {
        this.eraseTileByPoint(x, y);
        return;
      }
      // 添加马赛克
      this.drawTileByPoint(x, y);
    },
    //初始化马赛克（像素大小，刷子大小）；将原图分割为每个马赛克格子大小，将其数据保存于tile；
    initMosaic (context) {
      let { canvas } = context;
      const canvasWidth = canvas.width;
      const canvasHeight = canvas.height;
      const imageData = context.getImageData(0, 0, canvasWidth, canvasHeight).data;
      // 马赛克长宽
      let tileWidth = this.tileData.tileWidth;
      let tileHeight = this.tileData.tileHeight;
      // 马赛克总行数
      this.tileRowSize = Math.ceil(canvasHeight / tileHeight);
      // 马赛克总列数
      this.tileColumnSize = Math.ceil(canvasWidth / tileHeight);
      // 设置马赛克行列数
      let tiles = this.setTileRanks(this.tileRowSize, this.tileColumnSize, canvasWidth, canvasHeight);
      // 图片及原始图片中的像素（rgba）数据
      this.tiles = this.getPixelData(tiles, canvasWidth, imageData);
    },
    // 设置马赛克行列数,大小
    setTileRanks (rowSize, columnSize, canvasWidth, canvasHeight) {
      const tiles = [];
      for (let row = 0; row < rowSize; row++) {
        for (let column = 0; column < columnSize; column++) {
          //位置（行数，列数），大小
          const tile = {
            tileRow: row,
            tileColumn: column,
            pixelWidth: this.tileData.tileWidth,
            pixelHeight: this.tileData.tileHeight,
          }
          const { tileRow, tileColumn } = tile;
          //最后一行与最后一列
          if (column === columnSize - 1) {
            tile.pixelWidth = canvasWidth - column * tile.pixelWidth;
          }
          if (row === rowSize - 1) {
            tile.pixelHeight = canvasHeight - row * tile.pixelHeight;
          }
          tiles.push(tile);
        }
      }
      return tiles;
    },
    // 获取马赛克对应位置图片的r，g，b，a信息
    getPixelData (tiles, canvasWidth, imageData) {
      tiles.forEach((tile) => {
        // 图片信息（已添加马赛克）
        const imgTemData = []
        // 原始图片信息
        const initImageData = []
        for (let i = 0, j = tile.pixelHeight; i < j; i++) {
          //  马赛克左上角第一个像素点
          const pixelPosition = canvasWidth * this.tileData.tileHeight * tile.tileRow * 4 + tile.tileColumn * this.tileData.tileWidth * 4
          //  同一列的像素点
          const position = pixelPosition + canvasWidth * 4 * i

          imgTemData.push(...imageData.slice(position, position + tile.pixelWidth * 4))
          initImageData.push(...this.initImageData.slice(position, position + tile.pixelWidth * 4))
        }
        tile.imgData = imgTemData
        tile.initImageData = initImageData
      })
      return tiles
    },
    //增加马赛克
    drawTileByPoint (x, y) {
      //需要模糊处理的马赛克
      const drawTile = this.getTilesByPoints(x, y)
      this.drawMosaic(drawTile)
    },
    //全部填涂马赛克
    drawAllTiles () {
      this.drawMosaic(this.tiles)
    },
    //擦除马赛克
    eraseTileByPoint (x, y) {
      const clearTile = this.getTilesByPoints(x, y)
      this.eraseMosaic(clearTile)
    },
    //获取鼠标位置附近的马赛克位置
    getTilesByPoints (x, y) {
      //模糊处理的马赛克
      const drawTile = []
      let { tileWidth, tileHeight, brushSize } = this.tileData
      //鼠标(点)附近的马赛克（行数，列数）
      let startRow = Math.max(0, Math.floor(y / tileHeight) - Math.floor(brushSize / 2))
      let startColumn = Math.max(0, Math.floor(x / tileWidth) - Math.floor(brushSize / 2))
      let endRow = Math.min(this.tileRowSize, startRow + brushSize)
      let endColumn = Math.min(this.tileColumnSize, startColumn + brushSize)

      while (startRow < endRow) {
        let column = startColumn
        while (column < endColumn) {
          drawTile.push(this.tiles[startRow * this.tileColumnSize + column])
          column++
        }
        startRow++
      }
      return drawTile
    },
    //  通过修改图片rbga值来实现马赛克
    drawMosaic (drawTile) {
      let drawTiles = [].concat(drawTile)
      drawTiles.forEach(drawTile => {
        if (!drawTile.color) {
          let dataLen = drawTile.imgData.length
          let r = 0,
            g = 0,
            b = 0,
            a = 0
          for (let i = 0; i < dataLen; i += 4) {
            r += drawTile.imgData[i]
            g += drawTile.imgData[i + 1]
            b += drawTile.imgData[i + 2]
            a += drawTile.imgData[i + 3]
          }

          // 设置像素变化（平均值）
          let pixelLen = dataLen / 4
          drawTiles.color = {
            r: parseInt(r / pixelLen, 10),
            g: parseInt(g / pixelLen, 10),
            b: parseInt(b / pixelLen, 10),
            a: parseInt(a / pixelLen, 10)
          }
        }
        const color = drawTiles.color
        this.context.fillStyle = `rgba(${color.r}, ${color.g}, ${color.b}, ${color.a / 255})`

        const x = drawTile.tileColumn * this.tileData.tileWidth
        const y = drawTile.tileRow * this.tileData.tileHeight
        const width = drawTile.pixelWidth
        const height = drawTile.pixelHeight
        this.context.clearRect(x, y, width, height)
        this.context.fillRect(x, y, width, height)
      })
    },
    //清除马赛克
    eraseMosaic (clearTile) {
      let cleanTiles = [].concat(clearTile)
      cleanTiles.forEach(cleanTile => {
        const x = cleanTile.tileColumn * this.tileData.tileWidth
        const y = cleanTile.tileRow * this.tileData.tileHeight
        const width = cleanTile.pixelWidth
        const height = cleanTile.pixelHeight

        let imageData = this.context.createImageData(width, height)

        cleanTile.initImageData.forEach((val, i) => {
          imageData.data[i] = val
        })

        this.context.clearRect(x, y, width, height) // Clear.
        this.context.putImageData(imageData, x, y) // Draw.
      })
    },
    //清除所有马赛克
    eraseAllTiles () {
      this.eraseMosaic(this.tiles)
    },
    //提交，输出url
    submitCanvas (type, submitImage) {
      let img = this.canvas.toDataURL(type, submitImage)
      this.$emit("submitImage", img)
    },
    //设置刷子显隐，大小,位置
    setBrushPosition (show, target) {
      this.showBrush = show
      if (target) {
        let container = document.querySelector('.canvas-container-mosaic')
        let brushX = target.pageX - container.getBoundingClientRect().left - this.tileData.brushSize * this.tileData.tileWidth / 2
        let brushY = target.pageY - container.getBoundingClientRect().top - this.tileData.brushSize * this.tileData.tileHeight / 2
        this.brushStyle = {
          transform: `translate(${brushX}px,${brushY}px)`,
          width: this.tileData.brushSize * this.tileData.tileWidth + 'px',
          height: this.tileData.brushSize * this.tileData.tileHeight + 'px'
        }
      } else {
        this.brushStyle = {
          transform: `translate(0,0)`,
          width: 0,
          height: 0,
        }
      }
    },
  },
  watch: {
    tileData: {
      handler: function () {
        clearTimeout(this.timer)
        this.timer = null
        this.timer = setTimeout(() => {
          this.initMosaic(this.context)
        }, 50)
      },
      deep: true,
    }
  }
}
</script>
 
<style lang="less" scoped>
.drawMosaic {
  width: 100%;
  height: 100%;

  .canvas-container-mosaic {
    position: relative;
  }
  .mosaic-brush {
    position: absolute;
    z-index: 99;
    border: solid red 1px;
    top: 0;
    left: 0;
  }
}
</style>

<style>
/* .canvas-mosaic {
  border: solid black 2px;
} */
</style>

