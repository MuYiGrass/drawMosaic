<template>
  <div class="counter">
    <span class="label">{{label}} :</span>
    <button class="del"
            @click="changeCount('del')">
      <i>-</i>
    </button>
    {{count}}
    <button class="add"
            @click="changeCount('add')">
      +
    </button>
  </div>
</template>

<script>
export default {
  name: 'counter',
  props: {
    label: String,
    type: String,
    maxVal: Number,
    minVal: Number,
    defaultCount: Number
  },
  data () {
    return {
      count: this.defaultCount,
      timer: null
    }
  },
  methods: {
    changeCount (type, label) {
      if (type === 'add' && this.count < this.maxVal) {
        this.count++
      } else if (type === 'del' && this.count > this.minVal) {
        this.count--
      }
    }
  },
  watch: {
    count (newVal) {
      if (this.timer) {
        clearTimeout(this.timer)
      }
      this.timer = setTimeout(() => {
        this.$emit("changeCount", { count: this.count, type: this.type })
      }, 300)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" scoped>
.counter {
  font-weight: normal;
  font-size: 10px;
  padding: 5px;
  background-color: #f0f9eb;
  border-radius: 4px;
  margin: 5px;
  .label {
    margin-right: 2px;
  }
  button {
    background-color: #f0f9eb;
    color: #67c23a;
    border-radius: 4px;
    border: solid 1px #409eff;
  }
}
</style>
