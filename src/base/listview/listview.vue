<template>
  <scroll class="listview"
          ref="listview"
          :data="data"
          :listenScroll="listenScroll"
          :probeType="probeType"
          @scroll="scroll">
    <ul>
      <li v-for="group in data" class="list-group" ref="listGroup">
        <h2 class="list-group-title">{{group.title}}</h2>
        <ul>
          <li  v-for="item in group.items" class="list-group-item"
          @click="selectItem(item)">
            <img class="avatar" v-lazy="item.avatar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut"
         @touchstart.stop.prevent="onShortcutTouchStart"
         @touchmove.stop.prevent="onShortcutTouchMove">
      <ul>
        <li v-for="(item,index) in shortcutList" class="item" :data-index="index" :class="{'current':currentIndex===index}">{{item}}</li>
      </ul>
    </div>
    <div class="list-fixed" v-if="data.length" ref="fixed">
      <h1 class="fixed-title" v-show="fixedTitle">{{fixedTitle}}</h1>
    </div>
    <div class="loading-container" v-show="!data.length">
      <loading></loading>
    </div>
  </scroll>
</template>

<script type="text/ecmascript-6">
  import scroll from 'base/scroll/scroll'
  import loading from 'base/loading/loading'
  import {getData} from 'common/js/dom'
  const ANCHOR_HEIGHT = 18
  const TITLE_HEIGHT = 30

  export default {
    data () {
      return {
        scrollY: 0,
        currentIndex: 0,
        diff: 0
      }
    },
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    created () {
      this.listenScroll = true
      this.probeType = 3
      this.touch = {}
      this.listHeight = []
    },
    computed: {
      shortcutList () {
        return this.data.map((item) => {
          return item.title.substr(0, 1)
        })
      },
      fixedTitle () {
        if (this.scrollY > 0) {
          return ''
        }
        return this.data[this.currentIndex].title ? this.data[this.currentIndex].title : ''
      }
    },
    methods: {
      onShortcutTouchStart (ev) {
        let anchorIndex = parseInt(getData(ev.target, 'index'))
        let firstTouch = ev.touches[0]
        this.touch.y1 = firstTouch.pageY
        this.touch.anchorIndex = anchorIndex
        this._scrollTo(anchorIndex)
      },
      onShortcutTouchMove (ev) {
        let firstTouch = ev.touches[0]
        this.touch.y2 = firstTouch.pageY
        let offsetIndex = Math.floor((this.touch.y2 - this.touch.y1) / ANCHOR_HEIGHT)
        let anchorIndex = this.touch.anchorIndex + offsetIndex
        this._scrollTo(anchorIndex)
      },
      scroll (pos) {
        this.scrollY = pos.y
      },
      refresh () {
        this.$refs.listview.refresh()
      },
      _scrollTo (index) {
        if (!index && index !== 0) {    /* 点出边界处理 */
          return
        }
        if (index < 0) {            /* 滑出边界处理 */
          this.currentIndex = 0
        } else if (index > this.listHeight.length - 2) {
          this.currentIndex = this.listHeight.length - 2
        }
        this.scrollY = -this.listHeight[index]
        this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0)
      },
      _calculateHeight () {
        let list = this.$refs.listGroup
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < list.length; i++) {
          let item = list[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      },
      selectItem (item) {
        this.$emit('select', item)
      }
    },
    watch: {
      data () {
        setTimeout(() => {
          this._calculateHeight()
        }, 20)
      },
      scrollY (newY) {
        const listHeight = this.listHeight
        if (newY > 0) {             /* 滑出边界处理 */
          this.currentIndex = 0
          return
        }
        for (let i = 0; i < listHeight.length; i++) {
          let h1 = listHeight[i]
          let h2 = listHeight[i + 1]
          if (-newY >= h1 && -newY < h2) {
            this.currentIndex = i
            this.diff = h2 + newY
            return
          }
        }
        // 当滚动到底部，且-newY大于最后一个元素的上限
        this.currentIndex = listHeight.length - 2
      },
      diff (newval) {       // 是否到达临界点
        let fixedTop = (newval > 0 && newval < TITLE_HEIGHT) ? TITLE_HEIGHT - newval : 0
        if (this.fixedTop === fixedTop) {
          return
        }
        this.fixedTop = fixedTop
        this.$refs.fixed.style.transform = `translate(0,${-fixedTop}px)`
      }
    },
    components: {
      scroll,
      loading
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .listview
    position: relative
    width: 100%
    height: 100%
    overflow: hidden
    background: $color-background
    .list-group
      padding-bottom: 30px
      .list-group-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
      .list-group-item
        display: flex
        align-items: center
        padding: 20px 0 0 30px
        .avatar
          width: 50px
          height: 50px
          border-radius: 50%
        .name
          margin-left: 20px
          color: $color-text-l
          font-size: $font-size-medium
    .list-shortcut
      position: absolute
      z-index: 30
      right: 0
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding: 20px 0
      border-radius: 10px
      text-align: center
      background: $color-background-d
      font-family: Helvetica
      .item
        padding: 3px
        line-height: 1
        color: $color-text-l
        font-size: $font-size-small
        &.current
          color: $color-theme
    .list-fixed
      position: absolute
      top: 0
      left: 0
      width: 100%
      .fixed-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
    .loading-container
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>
