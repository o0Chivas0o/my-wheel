<template>
  <div class="w-slides">
    <div class="w-slides-window">
      <div class="w-slides-wrapper" ref="window">
        <slot></slot>
      </div>
    </div>
    <div class="w-slides-dots">
      <span v-for="n in childrenLength" :class="{active:selectedIndex === n-1}" @click="select(n-1)">
        {{n-1}}
      </span>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'WSlides',
    props: {
      selected: {type: String},
      autoPlay: {type: Boolean, default: true}
    },
    data () {
      return {
        childrenLength: 0,
        lastSelectedIndex: undefined
      }
    },
    computed: {
      selectedIndex () {
        return this.names.indexOf(this.selected) || 0
      },
      names () {
        return this.$children.map(vm => vm.name)
      }
    },
    mounted () {
      this.updateChildren()
      this.playAutomatically()
      this.childrenLength = this.$children.length
    },
    updated () {
      this.updateChildren()
    },
    methods: {
      playAutomatically () {
        let index = this.names.indexOf(this.getSelected())
        let run = () => {
          let newIndex = index - 1
          if (newIndex === -1) {newIndex = this.names.length - 1}
          if (newIndex === this.names.length) {newIndex = 0}
          this.select(newIndex)
          index++
          setTimeout(run, 3000)
        }
        // setTimeout(run, 3000)
      },
      select (index) {
        this.lastSelectedIndex = this.selectedIndex
        this.$emit('update:selected', this.names[index])
      },
      getSelected () {
        let first = this.$children[0]
        return this.selected || first.name
      },
      updateChildren () {
        let selected = this.getSelected()
        this.$children.forEach((vm) => {
          vm.reverse = this.selectedIndex <= this.lastSelectedIndex
          this.$nextTick(() => {
            vm.selected = selected
          })
        })
      }
    }
  }
</script>

<style lang="scss" scoped>
  .w-slides {
    &-window {overflow: hidden;}
    &-wrapper {position: relative;}
    &-dots {
      > span.active {background: red;}
    }
  }
</style>