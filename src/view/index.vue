<template>
  <price-step :degree="priceSections" :num="num" :unit="unit" :no-more="noMore" :add="0" :add-background="addBackground" :step-background="stepBackground"></price-step>
</template>

<script>
import priceStep from '@/components/segmentstep.vue'
export default {
  components: {
    priceStep
  },
  data () {
    return {
      priceSections: {
        60: [20, null],
        70: [12, 20],
        80: [6, 12],
        90: [3, 6]
      },
      num: 15,
      noMore: false,
      addBackground: '#0c9',
      stepBackground: '#f95f57',
      unit: { top: '元', bottom: '斤' }
    }
  }
}
</script>

<style>
</style>
