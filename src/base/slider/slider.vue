<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot>
      </slot>
    </div>
    <div class="dots">
      <span class="dot" v-for="(item,index) in dots"
            :class="{active:currentIndex===index}">
      </span>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import {addClass} from 'common/js/dom'
  import BScroll from 'better-scroll'

  export default {
    data () {
      return {
        dots: [],
        currentIndex: 0
      }
    },
    props: {
      loop: {
        type: Boolean,
        default: true
      },
      autoPlay: {
        type: Boolean,
        default: true
      },
      interval: {
        type: Number,
        default: 4000
      }
    },
    mounted () {
      setTimeout(() => {
        this.sliderGroup = this.$refs.sliderGroup
        this.children = this.sliderGroup.children
        this._initDots()
        this._setWidth()
        this._initScroll()

        if (this.autoPlay) {
          this._autoPlay()
        }

        window.addEventListener('resize', () => {
          if (!this.slider) {
            return
          }
          this._setWidth(true)
          this.slider.refresh()
        })
      }, 20)
    },
    methods: {
      _initDots () {
        for (let i = 0; i < this.children.length; i++) {
          this.dots.push([])
        }
      },
      _setWidth (resize) {
        let width = 0
        let slideWidth = this.$refs.slider.clientWidth
        for (let i = 0; i < this.children.length; i++) {
          addClass(this.children[i], 'slider-item')
          this.children[i].style.width = slideWidth + 'px'
        }
        width = this.children.length * slideWidth
        if (this.loop && !resize) {
          width += 2 * slideWidth
        }
        this.sliderGroup.style.width = width + 'px'
      },
      _initScroll () {
        this.slider = new BScroll(this.$refs.slider, {
          scrollX: true,
          scrollY: false,
          snap: true,
          momentum: false,
          snapLoop: this.loop,
          snapThreshold: 0.3,
          snapSpeed: 400,
          click: true
        })
        this.slider.on('scrollEnd', () => {
          let index = this.slider.getCurrentPage().pageX
          if (this.loop) {
            index--
          }
          this.currentIndex = index

          if (this.autoPlay) {
            this._autoPlay()
          }
        })
        this.slider.on('beforeScrollStart', () => {
          if (this.autoPlay) {
            clearTimeout(this.timer)
          }
        })
      },
      _autoPlay () {
        let index = this.currentIndex + 1
        if (this.loop) {
          index++
        }
        this.timer = setTimeout(() => {
          this.slider.goToPage(index, 0, 400)
        }, this.interval)
      }
    },
    activated () {
      if (this.autoPlay) {
        this._autoPlay()
      }
    },
    deactivated () {
      clearTimeout(this.timer)
    },
    beforeDestroy () {
      clearTimeout(this.timer)
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .slider
    min-height: 1px
    .slider-group
      position: relative
      overflow: hidden
      white-space: nowrap
      .slider-item
        float: left
        box-sizing: border-box
        overflow: hidden
        text-align: center
        a
          display: block
          width: 100%
          overflow: hidden
          text-decoration: none
        img
          display: block
          width: 100%
    .dots
      position: absolute
      right: 0
      left: 0
      bottom: 12px
      text-align: center
      font-size: 0
      .dot
        display: inline-block
        margin: 0 4px
        width: 8px
        height: 8px
        border-radius: 50%
        background: $color-text-l
        &.active
          width: 20px
          border-radius: 5px
          background: $color-text-ll
</style>
