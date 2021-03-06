<template>
  <div class="popover" ref="popover">
    <div class="content-wrapper" v-if="visible" ref="contentWrapper" :class="{[`position-${position}`]:true}">
      <slot name="content" :close="close"></slot>
    </div>
    <span ref="triggerWrapper" style="display: inline-block;">
      <slot></slot>
    </span>
  </div>
</template>

<script>
  export default {
    name: 'WPopover',
    data () {return {visible: false, timer: null}},
    props: {
      position: {
        type: String,
        default: 'top',
        validator (value) {
          return ['top', 'left', 'right', 'bottom'].indexOf(value) >= 0
        },
      },
      trigger: {
        type: String,
        default: 'click',
        validator (value) {
          return ['click', 'hover'].indexOf(value) >= 0
        }
      }
    },
    mounted () {
      if (this.trigger === 'click') {
        this.$refs.popover.addEventListener('click', this.onClick)
      } else {
        this.$refs.popover.addEventListener('mouseenter', this.hoverOpen)
        this.$refs.popover.addEventListener('mouseleave', this.hoverClose)
      }
    },
    beforeDestroy () {
      if (this.trigger === 'click') {
        this.$refs.popover.removeEventListener('click', this.onClick)
      } else {
        this.$refs.popover.removeEventListener('mouseenter', this.hoverOpen)
        this.$refs.popover.removeEventListener('mouseleave', this.hoverClose)
      }
    },
    methods: {
      onClick (event) {
        if (this.$refs.triggerWrapper.contains(event.target)) {
          if (!this.visible) {
            this.open()
          } else {
            this.close()
          }
        }
      },
      positionContent () {
        const {contentWrapper, triggerWrapper} = this.$refs
        document.body.appendChild(contentWrapper)
        const {width, height, top, left} = triggerWrapper.getBoundingClientRect()
        const {height: height2} = contentWrapper.getBoundingClientRect()
        let positions = {
          top: {top: top + window.scrollY, left: left + window.scrollX},
          bottom: {top: top + height + window.scrollY, left: left + window.scrollX},
          left: {top: top + window.scrollY + (height - height2) / 2, left: left + window.scrollX,},
          right: {top: top + window.scrollY + (height - height2) / 2, left: left + window.scrollX + width}
        }
        contentWrapper.style.left = positions[this.position].left + 'px'
        contentWrapper.style.top = positions[this.position].top + 'px'
      },
      onClickDocument (e) {
        if (this.$refs.popover && (this.$refs.popover.contains(e.target) || this.$refs.popover === e.target)) {return}
        if (this.$refs.contentWrapper && (this.$refs.contentWrapper.contains(e.target) || this.$refs.contentWrapper === e.target)) {return}
        this.close()
      },
      hoverOpen () {
        this.clearTimer()
        this.visible = true
        this.$nextTick(() => {
          const {contentWrapper} = this.$refs
          this.positionContent()
          contentWrapper.addEventListener('mouseenter', this.hoverOpen)
          contentWrapper.addEventListener('mouseleave', this.hoverClose)
        })
      },
      hoverClose () {
        const {contentWrapper} = this.$refs
        this.timer = setTimeout(() => {
          this.visible = false
          this.timer = null
          contentWrapper.removeEventListener('mouseenter', this.open)
          contentWrapper.removeEventListener('mouseleave', this.close)
        }, 200)
      },
      open () {
        this.visible = !this.visible
        this.$nextTick(() => {
          this.positionContent()
          document.addEventListener('click', this.onClickDocument)
        })
      },
      close () {
        this.visible = false
        document.removeEventListener('click', this.onClickDocument)
      },
      clearTimer () {
        if (this.timer) {
          clearTimeout(this.timer)
          this.timer = null
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import '../style/var';
  
  .popover {display: inline-block;vertical-align: top;position: relative;}
  .content-wrapper {
    position: absolute;background: white; /*box-shadow: 0 0 3px rgba(0, 0, 0, .5);*/
    filter: drop-shadow(0 1px 1px rgba(0, 0, 0, .5));
    border: 1px solid $border-color;border-radius: $border-radius;padding: .5em 1em;max-width: 24em;word-break: break-all;
    &::before, &::after {content: '';display: block;width: 0;height: 0;border: 10px solid transparent;position: absolute;}
    &.position-top {
      margin-top: -10px;
      transform: translateY(-100%);
      &::before, &::after {left: 10px;border-bottom: none;}
      &::before {top: 100%;border-top-color: black;}
      &::after {top: calc(100% - 1px);border-top-color: white;}
    }
    &.position-bottom {
      margin-top: 10px;
      &::before, &::after {left: 10px;border-top: none;}
      &::before {bottom: 100%;border-bottom-color: black;}
      &::after {bottom: calc(100% - 1px);border-bottom-color: white;}
    }
    &.position-left {
      margin-left: -10px;
      transform: translateX(-100%);
      &::before, &::after {left: 100%;top: 50%;transform: translateY(-50%);border-right: none;}
      &::before {border-left-color: black;}
      &::after {border-left-color: white;left: calc(100% - 1px);}
    }
    &.position-right {
      margin-left: 10px;
      &::before, &::after {right: 100%;top: 50%;transform: translateY(-50%);border-left: none;}
      &::before {border-right-color: black;}
      &::after {border-right-color: white;right: calc(100% - 1px);}
    }
  }
</style>
