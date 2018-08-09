<template>
  <div class="wv-tab-container-item" :style="style">
    <slot />
  </div>
</template>

<script>
import { create } from '../../utils'

export default create({
  name: 'wv-tab-container-item',

  data () {
    return {
      offset: 0
    }
  },

  computed: {
    style () {
      return {
        width: this.$parent.width + 'px',
        transform: `translate3d(${this.offset}px, 0, 0)`
      }
    }
  },

  beforeCreate () {
    this.$parent && this.$parent.pages.push(this)
  },

  destroyed () {
    this.$parent && this.$parent.pages.splice(this.$parent.pages.indexOf(this), 1)
  }
})
</script>

<style scoped lang="scss">
  .wv-tab-container-item {
    float: left;
    height: 100%;
  }
</style>
