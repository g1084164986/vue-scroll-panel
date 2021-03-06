<template>
  <div class="vue-scroll-root" ref="root">
    <div class="vue-scroll-outer" ref="out">
      <div class="vue-scroll-inner" ref="in">
        <slot></slot>
      </div>
    </div>
    <slot name="more">
      <div class="vue-scroll-more-icon" ref="icon" @click="moreButtonClick">
        <svg xmlns="http://www.w3.org/2000/svg"
              width="24"
              height="24"
              viewBox="0 0 24 24"
              :class="{expand}">
          <path d="M16.59 8.59L12 13.17 7.41 8.59 6 10l6 6 6-6z"/>
          <path d="M0 0h24v24H0z" fill="none"/>
        </svg>
      </div>
    </slot>
  </div>
</template>

<script>
import animateScrollTo from 'animated-scroll-to'
import ScrollBooster from 'scrollbooster'

import "./style/main.less"

export default {
  data() {
    return {
      expand: false,
    }
  },
  props: {
    scroll: {
      type: [HTMLElement, Number]
    },
    dragable: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    scrollTo(val){
      animateScrollTo(val,
        {
          elementToScroll: this.expand ? this.$refs.in : this.$refs.out
        })
      .then(() => {
        if(!(val instanceof HTMLElement)) return
        let ori = val.style.boxShadow
        val.style.boxShadow = '0 0 0 2px #FFB11BAA'
        setTimeout(() => {
          val.style.boxShadow = ori
        }, 1000)
      })
    },
    moreButtonClick(){
      if(!this.expand) this.expandPanel()
      else this.collapsePanel()
    },
    expandPanel(){
      this.$refs.root.style.height = `${this.$refs.root.clientHeight}px`
      this.$refs.in.style.width = `${this.$refs.out.clientWidth}px`
      this.$refs.icon.style.left = `${this.$refs.out.clientWidth}px`
      this.$refs.out.style.overflow = `unset`
      this.$refs.in.style.whiteSpace = `normal`
      // this.$refs.in.style.border = `1px #6c757d solid`
      this.$refs.in.style.position = `absolute`
      this.$refs.in.style.backgroundColor = `white`
      this.$refs.in.style.maxHeight = `500px`
      this.$refs.in.style.overflow = `auto`
      this.$refs.icon.style.position = `relative`
      this.expand = true
      this.$emit("panel-expand")
    },
    collapsePanel(){
      this.$refs.root.style.height = ''
      this.$refs.in.style.width = ''
      this.$refs.icon.style.left = ``
      this.$refs.out.style.overflow = ``
      this.$refs.in.style.whiteSpace = ''
      this.$refs.in.style.border = ``
      this.$refs.in.style.position = ``
      this.$refs.in.style.backgroundColor = ``
      this.$refs.in.style.maxHeight = ``
      this.$refs.in.style.overflow = ``
      this.$refs.icon.style.position = ``
      this.expand = false
    }
  },
  mounted() {
    this.$on('scroll-left', () => {
      if(this.expand) return
      let distance = this.$refs.out.scrollLeft - this.$refs.out.clientWidth * .8
      this.scrollTo([distance, 0])
    })
    this.$on('scroll-right', () => {
      if(this.expand) return
      let distance = this.$refs.out.scrollLeft + this.$refs.out.clientWidth * .8
      this.scrollTo([distance, 0])
    })
    this.$on('expand-panel', () => {
      this.expandPanel()
    })

    // drag to scroll
    if(this.dragable)this.$nextTick(() => {
      let viewport = this.$refs.out
      let content = this.$refs.in
      let sb = new ScrollBooster({
        viewport,
        content,
        onUpdate: (data)=> {
          if(!this.dragable) return;
          // this.$emit('scroll', data)
          if(!data.isScrolling) viewport.scrollLeft = data.position.x
        }
      })

    })
  },
  watch: {
    scroll: function (newVal) {
      this.scrollTo(newVal)
    }
  },
}
</script>

<style>
</style>
