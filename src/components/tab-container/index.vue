<template>
  <div
    class="wv-tab-container"
    @touchstart="onTouchstart"
    @touchmove="onTouchmove"
    @touchend="onTouchend"
    @touchcancel="onTouchend">
    <div
      :style="wrapperStyle"
      class="wv-tab-container__wrapper"
    >
      <slot/>
    </div>
  </div>
</template>

<script>
import { create, getTouch } from '../../utils'

export default create({
  name: 'wv-tab-container',

  props: {
    height: Number,
    defaultIndex: {
      type: Number,
      default: 0
    },
    duration: {
      type: Number,
      default: 500
    },
    noDragWhenSingle: {
      type: Boolean,
      default: true
    }
  },

  data () {
    return {
      width: 0,
      offset: 0,
      startX: 0,
      startY: 0,
      active: 0,
      deltaX: 0,
      pages: [],
      direction: '',
      currentDuration: 0
    }
  },

  mounted () {
    this.initialize()
  },

  destroyed () {
    clearTimeout(this.timer)
  },

  watch: {
    pages () {
      this.initialize()
    },

    defaultIndex () {
      this.initialize()
    }
  },

  computed: {
    count () {
      return this.pages.length
    },

    wrapperStyle () {
      let ret = {
        paddingLeft: this.count > 1 ? this.width + 'px' : 0,
        width: this.count > 1 ? (this.count + 2) * this.width + 'px' : '100%',
        transitionDuration: `${this.currentDuration}ms`,
        transform: `translate3d(${this.offset}px, 0, 0)`
      }

      if (this.height) {
        ret.height = this.height + 'px'
      }

      return ret
    }
  },

  methods: {
    initialize () {
      clearTimeout(this.timer)
      this.width = this.$el.getBoundingClientRect().width
      this.active = this.defaultIndex
      this.currentDuration = 0
      this.offset = this.count > 1 ? -this.width * (this.active + 1) : 0
      this.pages.forEach(swipe => {
        swipe.offset = 0
      })
    },

    onTouchstart (event) {
      if (this.count === 1 && this.noDragWhenSingle) {
        return
      }

      clearTimeout(this.timer)
      const touch = getTouch(event)

      this.deltaX = 0
      this.direction = ''
      this.currentDuration = 0
      this.startX = touch.clientX
      this.startY = touch.clientY

      if (this.active <= -1) {
        this.move(this.count)
      }
      if (this.active >= this.count) {
        this.move(-this.count)
      }
    },

    onTouchmove (event) {
      event.preventDefault()

      const touch = getTouch(event)

      this.deltaX = touch.clientX - this.startX
      const deltaY = touch.clientY - this.startY

      if (this.count === 1) {
        if (this.noDragWhenSingle) return

        this.offset = this.range(this.deltaX, [-20, 20])
      } else if (this.count > 1 && Math.abs(this.deltaX) > Math.abs(deltaY)) {
        this.move(0, this.range(this.deltaX, [-this.width, this.width]))
      }
    },

    onTouchend () {
      if (this.count === 1) {
        if (this.noDragWhenSingle) return

        this.offset = 0
        this.currentDuration = this.duration
      } else {
        if (this.deltaX) {
          this.move(Math.abs(this.deltaX) > 50 ? (this.deltaX > 0 ? -1 : 1) : 0)
          this.currentDuration = this.duration
        }
      }
    },

    move (move = 0, offset = 0) {
      const {active, count, pages, deltaX, width} = this

      if (move) {
        if (active === -1) {
          pages[count - 1].offset = 0
        }
        pages[0].offset = active === count - 1 && move > 0 ? count * width : 0

        this.active += move
      } else {
        if (active === 0) {
          pages[count - 1].offset = deltaX > 0 ? -count * width : 0
        } else if (active === count - 1) {
          pages[0].offset = deltaX < 0 ? count * width : 0
        }
      }
      this.offset = offset - (this.active + 1) * this.width
    },

    range (num, arr) {
      return Math.min(Math.max(num, arr[0]), arr[1])
    }
  }
})
</script>

<style scoped lang="scss">
  .wv-tab-container {
    overflow: hidden;
    position: relative;
    user-select: none;

    &__wrapper {
      height: 100%;
      overflow: hidden;
    }
  }
</style>
