<template>
  <div class="w-slides"
       @touchstart="onTouchStart" @touchend="onTouchEnd"
       @mouseenter="onMouseEnter" @mouseleave="onMouseLeave">
    <div class="w-slides-window">
      <div class="w-slides-wrapper" ref="window">
        <slot></slot>
      </div>
    </div>
    <div class="w-slides-dots">
      <span @click="onClickPrev" data-action="prev">
        <w-icon name="left"></w-icon>
      </span>
      <span v-for="n in childrenLength" :class="{active:selectedIndex === n-1}" :key="n"
            @click="select(n-1)"
            :data-index="n-1">
        {{n}}
      </span>
      <span @click="onClickNext" data-action="next">
        <w-icon name="right"></w-icon>
      </span>
    </div>
  </div>
</template>

<script>
  import WIcon from '../icon/icon'
  
  export default {
    name: 'WSlides',
    components: {WIcon},
    props: {
      selected: {type: String},
      autoPlay: {type: Boolean, default: true},
      autoPlayDelay: {type: Number, default: 3000}
    },
    data () {
      return {
        childrenLength: 0,
        lastSelectedIndex: undefined,
        timerId: undefined,
        startTouch: undefined
      }
    },
    computed: {
      selectedIndex () {
        let index = this.names.indexOf(this.selected)
        return index === -1 ? 0 : index
      },
      names () {
        return this.items.map(vm => vm.name)
      },
      items () {
        return this.$children.filter(vm => vm.$options.name === 'WSlidesItem')
      }
    },
    mounted () {
      this.updateChildren()
      if (this.autoPlay) {this.playAutomatically()}
      this.childrenLength = this.items.length
    },
    updated () {
      this.updateChildren()
    },
    methods: {
      onClickPrev () {
        this.select(this.selectedIndex - 1)
      },
      onClickNext () {
        this.select(this.selectedIndex + 1)
      },
      onTouchStart (e) {
        if (e.touches.length > 1) {return}
        this.startTouch = e.touches[0]
        this.pause()
      },
      onTouchEnd (e) {
        let {clientX: x1, clientY: y1} = this.startTouch
        let {clientX: x2, clientY: y2} = e.changedTouches[0]
        let distance = Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2))
        let deltaY = Math.abs(y2 - y1)
        let rate = distance / deltaY // 检测用户是否要切换slide
        if (rate > 2) {
          if (x2 > x1) {
            this.select(this.selectedIndex - 1)   // 右
          } else {
            this.select(this.selectedIndex + 1)   // 左
          }
        } else {
          return
        }
        this.$nextTick(() => {
          this.playAutomatically()
        })
      },
      onMouseEnter () {
        this.pause()
      },
      onMouseLeave () {
        this.playAutomatically()
      },
      playAutomatically () {
        if (this.timerId) {return}
        let run = () => {
          let index = this.names.indexOf(this.getSelected())
          let newIndex = index + 1
          this.select(newIndex) // 告诉外界选中 newIndex
          this.timerId = setTimeout(run, this.autoPlayDelay)
        }
        this.timerId = setTimeout(run, this.autoPlayDelay)
      },
      pause () {
        window.clearTimeout(this.timerId)
        this.timerId = undefined
      },
      select (index) {
        this.lastSelectedIndex = this.selectedIndex
        if (index === -1) {index = this.names.length - 1}
        if (index === this.names.length) {index = 0}
        this.$emit('update:selected', this.names[index])
      },
      updateChildren () {
        let selected = this.getSelected()
        this.items.forEach((vm) => {
          let reverse = this.selectedIndex <= this.lastSelectedIndex
          if (this.timerId) {
            if (this.lastSelectedIndex === this.items.length - 1 && this.selectedIndex === 0) {
              reverse = false
            }
            if (this.lastSelectedIndex === 0 && this.selectedIndex === this.items.length - 1) {
              reverse = true
            }
          }
          vm.reverse = reverse
          this.$nextTick(() => {
            vm.selected = selected
          })
        })
      },
      getSelected () {
        let first = this.items[0]
        return this.selected || first.name
      }
    }
  }
</script>

<style lang="scss" scoped>
  .w-slides {
    &-window {overflow: hidden;}
    &-wrapper {position: relative;}
    &-dots {
      padding: 8px 0;display: flex;justify-content: center;align-items: center;
      > span {
        width: 24px;height: 24px;border-radius: 50%;margin: 0 4px;font-size: 16px;
        display: inline-flex;background: #dddddd;justify-content: center;align-items: center;color: white;fill: white;
        &:hover {cursor: pointer;}
        &.active {
          background: #000000;
          &:hover {cursor: default;}
        }
      }
    }
  }
</style>
