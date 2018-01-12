<!--

# inputText
# type => input type 使用： :value.aync = 'example'
# value => input value
# size => input size
# placeholder => input tip
# disabled => 是否可用
# max => 最大字数
# sufFixIcon: 后置icon 十六进制数 example:

-->
<template>
  <div :class="[
    type === 'textarea' ? 'yd-textarea' : 'yd-input',
    {
      'is-disabled': disabled,
      'yd-input__group_append': $slots.prepend || $slots.append,
    }
    ]">
    <template v-if="type !== 'textarea'">
      <div class="yd-input__wrap">
        <div class="yd-input__group_prepend" v-if="$slots.prepend">
          <slot name="prepend"></slot>
        </div>
        <div class="yd-input__main">
          <input
            ref="input"
            v-bind="$props"
            :class="[
              size ? `yd-input__size_${size}` : '',
              'yd-input__inner',
              (max || sufFixIcon) && type != 'number' ? 'yd-input__inner_icon' : '',
            ]"
            :id="id"
            :type="type"
            :placeholder="placeholder"
            :value="currentValue"
            @input="handleInput"
            @blur="handleBlur"
            @change="handleChange"
            @keyup.enter="handleEnter"
            @keyup="handleKeyup"
            @keypress="handleKeypress"
            @keydown="handleKeydown"
            @mousedown="handleMouseDown"
            @click="handleClick"
            >
          <div class="yd-input__icon" v-if="max || sufFixIcon">
            <span v-if="max && type != 'number'">{{Math.floor(currentValue.length/2)}}/{{max}}</span>
            <i v-if="!max && sufFixIcon && type != 'number'" @click="handleIconClick" v-html="sufFixIcon"></i>
          </div>
        </div>
        <div class="yd-input__group_append" v-if="$slots.append">
          <slot name="append"></slot>
        </div>
      </div>
    </template>
    <template v-else>
      <div class="yd-textarea-wrap">
        <textarea
          class="yd-textarea__inner"
          :placeholder="placeholder"
          :id="id"
          :rows="rows"
          :value="currentValue"
          @input="handleInput"
          @blur="handleBlur"
          @change="handleChange"
          @keyup.enter="handleEnter"
          >
          </textarea>
      </div>
    </template>
  </div>
</template>

<script>
import { oneOf } from 'UI/utils/util'
export default {
  name: 'YdInput',
  props: {
    type: {
      validator(val) {
        return oneOf(val, ['text', 'textarea', 'number', 'password', 'url', 'email'])
      },
      default: ''
    },
    value: {
      type: [String, Number],
      default: ''
    },
    autosize: {
      type: [Boolean, Object],
      default: false
    },
    size: {
      validator(val) {
        return oneOf(val, ['small', 'large', 'default'])
      },
      default: 'default'
    },
    placeholder: {
      type: String,
      value: ''
    },
    max: Number,
    id: String,
    disabled: {
      type: Boolean,
      default: false
    },
    //后置图标
    sufFixIcon: String,
    rows: {
      type: Number,
      value: 2
    },
    name: String,
    autofocus: {
      type: Boolean,
      default: false
    },
    autoComplete: {
      validator(val) {
        return oneOf(val, ['off', 'on'])
      },
      default: 'off'
    },
    spellcheck: {
      type: Boolean,
      default: false
    },
    readonly: Boolean
  },
  data() {
    return {
      currentValue: this.value,
      currentLen: 0
    }
  },
  watch: {
    autofocus: function(val) {
      if(val) this.autoFocus()
    }
  },
  methods: {
    autoFocus() {
      (this.$refs.input || this.$refs.textarea).focus()
    },
    handleMouseDown(e) {
      this.$emit('mousedown', e)
    },
    handleEnter(e) {
      this.$emit('enter', e)
    },
    handleBlur(e) {
      this.$emit('blur', e);
    },
    handleKeydown(event) {
      this.$emit('keydown', event);
    },
    handleMousedown(event) {
      this.$emit('mousedown', event);
    },
    handleKeypress(event) {
      this.$emit('keypress', event);
    },
    handleKeyup (event) {
      this.$emit('keyup', event);
    },
    handleChange(event) {
      this.$emit('change', event);
    },
    handleIconClick(event) {
      this.$emit('iconClick', event)
    },
    setCurrentValue(value) {
      this.currentValue = value;
    },
    handleInput(e) {
      let value = e.target.value;
      this.$emit('input', value);
      this.$emit('update:value', value);
      this.setCurrentValue(value);
    },
    handleClick(e) {
      this.$emit('click', e)
    }
  },
  watch: {
    currentValue(val, old) {
      if (this.max) {
        let bitLen = val.replace(/[^x00-xff]/ig, '**').length, sliceLen = '';
        if (bitLen > (this.max*2)) {
          this.currentLen = this.max*2;
          sliceLen = val.slice(0, this.currentLen).replace(/\*\*/g, '&').replace(/\*/g, '').length;
          this.$nextTick(() => {
            this.currentValue = val.slice(0, sliceLen)
          })
        }
      }
    },
    value(val, old) {
      this.setCurrentValue(val)
    }
  }
}
</script>
<style media="screen" lang="stylus">
$baseColor = #da3838
$baseLightColor = #e8a5a5

borderRadius(n)
  -webkit-border-radius n
  -moz-border-radius n
  border-radius n
.is-disabled
  cursor not-allowed
  background-color #eef1f6
  border-color #d1dbe5
  color #bbb
  input
    background-color #f5f7fa
    border-color #e4e7ed
    color #c0c4cc
    cursor not-allowed

.yd-input
  position relative
  font-size 14px
  display inline-block
  width 100%
  .yd-input__main
    position relative
  .yd-input__inner
    position relative
    -webkit-appearance none
    background-color #fff
    background-image none
    borderRadius(4px)
    border 1px solid #dcdfe6
    box-sizing border-box
    color #606266
    display inline-block
    font-size inherit
    height 40px
    line-height 1
    outline none
    padding 0 15px
    transition border-color .2s cubic-bezier(.645,.045,.355,1)
    width 100%
    &:hover, &:visited
      border 1px solid $baseLightColor !important
  .yd-input__size_default
    height 38px
  .yd-input__size_large
    height 42px
  .yd-input__size_small
    height 32px
  .yd-input__inner_icon
    padding-right 50px
  .yd-input__icon
    width 42px
    text-align center
    padding 0 4px
    position absolute
    top 0
    right 0
    height 100%
    color #606266
    i
    span
      display flex
      height 100%
      justify-content center
      vertical-align middle
      align-items center
      overflow hidden
.yd-input__group_append
  .yd-input__main
    input
      borderRadius(0)
  .yd-input__wrap
    width 100%
    display inline-table
    div
      display table-cell
    .yd-input__group_prepend
    .yd-input__group_append
      width 1px
      padding 0px 20px
      border 1px solid #dcdfe6
      background-color #f5f7fa
      color #606266
      font-size inherit
      transition border-color .2s cubic-bezier(.645,.045,.355,1)
    .yd-input__group_prepend
      border-right 0
    .yd-input__group_append
      border-left 0
.yd-textarea
  position relative
  font-size 14px
  display inline-block
  width 100%
  .yd-textarea__inner
    outline:none
    display block
    resize vertical
    padding 5px 15px
    line-height 1.5
    box-sizing border-box
    width 100%
    font-size inherit
    color #606266
    background-color #fff
    background-image none
    border 1px solid #dcdfe6
    border-radius 4px
    transition border-color .2s cubic-bezier(.645,.045,.355,1)
    &:hover, &:visited
      border 2px solid $baseLightColor !important
</style>
