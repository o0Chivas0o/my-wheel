<template>
  <div class="wrapper" :class="{error}">
    <input type="text" :value="value" :disabled="disabled" :readonly="readonly"
           @change="$emit('change',$event.target.value)"
           @input="$emit('input',$event.target.value)"
           @focus="$emit('focus',$event.target.value)"
           @blur="$emit('blur',$event.target.value)"
    >
    <template v-if="error">
      <Icon name="error" class="icon-error"></Icon>
      <span class="error-message">{{error}}</span>
    </template>
  </div>
</template>
<script>
  import Icon from '@/icon/icon'
  
  export default {
    name: 'WInput',
    components: {Icon},
    props: {
      value: {type: String},
      disabled: {type: Boolean, default: false},
      readonly: {type: Boolean, default: false},
      error: {type: String}
    }
  }
</script>
<style lang="scss" scoped>
  @import '../style/var';
  
  .wrapper {
    font-size: $font-size;display: inline-flex;align-items: center;
    > :not(:last-child) {margin-right: .5em;}
    > input {
      height: $height;border: 1px solid $input-border-color;border-radius: 4px;font-size: inherit;padding: 0 8px;
      &:hover {border-color: $border-color-hover;}
      &:focus {box-shadow: inset 0 1px 3px $box-shadow-color;outline: none;}
      &[disabled], &[readonly] {border-color: #bbb;color: #bbb;cursor: not-allowed;}
    }
    &.error {
      > input {border-color: $red;}
    }
    .icon-error {fill: $red;}
    .error-message {color: $red;}
  }
</style>
