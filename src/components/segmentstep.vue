<template>
  <div class="step-container">
    <div class="line">
      <div class="progress-top" :class="{end:noMore}">
        <div v-for="(item,index) in list" :key="index" class="item"
          :class='{active:degreeList[index]!==0,through:throughList[index]}'>￥{{item}}
        </div>
      </div>
      <div class="progress-middle">
        <!-- 已选数量但是没有确定的时候，显示绿色 -->
        <div class="progress-add" :style="{width:addPercent+'%',background: addBackground}"></div>
        <div class="progress-step" :style="{width:percent+'%',background: stepBackground}"></div>
        <div v-for="(degree,dIndex) in degreeList" :key="dIndex" class="circle-box" :class="{end:noMore}">
          <div class="circle" :class="{activeNum:degree===1,activeAdd:degree===2,end:noMore}"></div>
        </div>
      </div>
      <div class="progress-bottom">
        <div v-for="(data,dIndex) in dataList" :key="dIndex" class="weight-item" :class="{end:noMore}">
          {{data}}
        </div>
      </div>
    </div>
    <div class="unit">
      <div>({{unit.top}})</div>
      <div>({{unit.bottom}})</div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    noMore: {
      type: Boolean,
      default: false
    },
    add: {
      type: Number,
      default: 0
    },
    num: {
      type: Number,
      default: 0
    },
    degree: {
      type: Object,
      default: () => {}
    },
    addBackground: {
      type: String,
      default: ''
    },
    stepBackground: {
      type: String,
      default: ''
    },
    unit: {
      type: Object,
      default: () => {
        return {
          top: '',
          bottom: ''
        }
      }
    }
  },
  data () {
    return {
      list: [],
      dataList: [],
      percent: 0,
      addPercent: 0,
      throughList: [],
      degreeList: []
    }
  },
  mounted () {
    this.computeAddStep()
    this.computeNumStep()
  },
  methods: {
    // 已选数量和确定的数量要分开计算，否则会影响进度条长度变化
    computeNumStep () {
      if (!this.degree) return
      let num = parseInt(this.num)
      let prices = Object.keys(this.degree).reverse().map(item => {
        return parseFloat(item)
      })
      let scale = parseInt(100 / prices.length)
      let steps = Object.values(this.degree).reverse()
      let minWeight = parseInt(steps[0][0])
      let maxWeight = parseInt(steps[steps.length - 1][0])
      let that = this
      for (let i = 0; i < prices.length; i++) {
        this.list[i] = prices[i]
        this.dataList[i] = parseInt(steps[i][0])
        this.degreeList[i] = 0
        this.throughList[i] = false
      }
      let percent = num - minWeight > maxWeight ? maxWeight : num - minWeight
      that.percent = parseFloat(percent / maxWeight).toFixed(2) * 100
      let len = prices.length
      for (let d = 0; d < len; d++) {
        if (num === 0) {
          return
        } else if (num > 0 && num <= steps[0][0]) { // 数量大于0不到最低数量时
          that.degreeList[0] = 1
        } else if (num === steps[d][1] && steps[d][1]) { // 数量刚好等于节点值
          that.percent = scale * (d + 1) // 由于各级差值不同，导致比例不同，采用等比例的方式
          for (let n = 0; n <= d + 1; n++) {
            that.degreeList[n] = 1
            if (n > 0) {
              that.throughList[n - 1] = true
            }
          }
        } else if (num > steps[d][0] && num <= steps[d][1] && steps[d][1]) { // 数量在两个节点之间
          that.percent = scale * d + (num - steps[d][0]) / (steps[d][1] - steps[d][0]) * scale // 按照每个阶段的比例划线的长度
          for (let n = 0; n < d + 1; n++) {
            that.degreeList[n] = 1
            if (n > 0) {
              that.throughList[n - 1] = true
            }
          }
        } else if (!steps[d][1] && num >= steps[d][0]) { // 总数量刚好达到最后一档或节点超出最大值
          that.percent = 100
          for (let n = 0; n < d + 1; n++) {
            that.degreeList[n] = 1
            if (n > 0) {
              that.throughList[n - 1] = true
            }
          }
        }
      }
    },
    // 动态修改用户选择数量后的进度条颜色及长度
    computeAddStep () {
      if (!this.degree) return
      let add = this.add === 0 ? 0 : parseInt(this.add) + parseInt(this.num)
      let prices = Object.keys(this.degree).reverse().map(item => {
        return parseFloat(item)
      })
      let scale = parseInt(100 / prices.length)
      let steps = Object.values(this.degree).reverse()
      let minWeight = parseInt(steps[0][0])
      let maxWeight = parseInt(steps[steps.length - 1][0])
      let that = this
      for (let i = 0; i < prices.length; i++) {
        that.list[i] = prices[i]
        that.dataList[i] = parseInt(steps[i][0])
      }
      that.addPercent = add - minWeight > maxWeight ? maxWeight : add - minWeight
      let len = prices.length
      for (let d = 0; d < len; d++) {
        if (add === 0) {
          return
        } else if (add > 0 && add <= steps[0][0]) { // 数量大于0不到最低数量时
          that.degreeList[0] = 2
        } else if (add === steps[d][1] && steps[d][1]) { // 数量刚好等于节点值
          that.addPercent = scale * d + (add - steps[d][0]) / (steps[d][1] - steps[d][0]) * scale // 按照每个阶段的比例划线的长度
          let _index = that.data.degreeList.lastIndexOf(1) + 1
          for (let n = _index; n <= d + 1; n++) {
            that.degreeList[n] = 2
            if (n > 0) {
              that.throughList[n - 1] = true
            }
          }
          for (let m = d + 1; m < len; m++) {
            that.degreeList[m] = 0
            if (m > 0) { // 减少选择数量时，将上面的样式修改掉
              that.throughList[m - 1] = false
            }
          }
        } else if (add >= steps[d][0] && add < steps[d][1] && steps[d][1]) { // 数量在两个节点之间
          that.addPercent = scale * d + (add - steps[d][0]) / (steps[d][1] - steps[d][0]) * scale // 按照每个阶段的比例划线的长度
          let _index = that.data.degreeList.lastIndexOf(1) + 1
          for (let n = _index; n < d + 1; n++) {
            that.degreeList[n] = 2
            if (n > 0) {
              that.throughList[n - 1] = true
            }
          }
          for (let m = d + 1; m < len; m++) {
            that.degreeList[m] = 0
            if (m > 0) { // 减少选择数量时，将上面的样式修改掉
              that.throughList[m - 1] = true
            }
          }
        } else if (!steps[d][1] && add >= steps[d][0]) { // 总数量刚好达到最后一档或节点超出最大值
          let _index = that.data.degreeList.lastIndexOf(1) + 1
          for (let n = _index; n < d + 1; n++) {
            that.degreeList[n] = 2
            if (n > 0) {
              that.throughList[n - 1] = true
            }
          }
          for (let m = d + 1; m < len; m++) {
            that.degreeList[m] = 0
            if (m > 0) { // 减少选择数量时，将上面的样式修改掉
              that.throughList[m - 1] = false
            }
          }
        }
      }
    }
  }
}
</script>

<style>
  .step-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    position: relative;
  }

  .line {
    width: 80%;
  }

  .unit {
    flex-direction: row;
    position: absolute;
    height: 100%;
    right: 0;
    display: flex;
    flex-flow: column;
    justify-content: space-between;
  }

  .unit view:nth-child(2) {
    position: absolute;
    bottom: 0;
  }

  .progress-top,
  .progress-middle,
  .progress-bottom {
    display: flex;
  }

  .progress-top {
    font-size: 12px;
    justify-content: space-around;
  }

  .progress-top.end {
    justify-content: space-between;
  }

  .progress-middle,
  .progress-bottom {
    justify-content: space-between;
  }

  .item.active {
    color: #f95f57;
  }

  .item.through {
    text-decoration: line-through;
  }

  .progress-middle {
    margin: 10px 0;
    width: 100%;
    height: 4px;
    background: #cfcccf;
    border-radius: 4px;
    position: relative;
  }

  .progress-add {
    position: absolute;
    height: 4px;
    z-index: 0;
    border-radius: 4px 0 0 4px;
    background: #0c9;
  }

  .progress-step {
    position: absolute;
    height: 4px;
    background: #f95f57;
    z-index: 0;
    border-radius: 4px 0 0 4px;
    flex-wrap: nowrap;
  }

  .circle-box,
  .weight-item {
    flex: 1;
    text-align: left;
    z-index: 1;
  }

  .circle-box.end,
  .weight-item.end {
    text-align: center;
    flex: 0;
  }

  .circle {
    z-index: 1;
    width: 8px;
    height: 8px;
    background: #cfcccf;
    border-radius: 50%;
    margin-top: -2px;
    margin-left: -4px;
  }

  .circle.end {
    margin-left: 0;
  }

  .circle.activeNum {
    background: #f95f57;
  }

  .circle.activeAdd {
    background: #0c9;
  }

  .weight-item {
    font-size: 12px;
    margin-left: -4px;
  }

  .weight-item.end {
    margin-left: 0;
  }
</style>
