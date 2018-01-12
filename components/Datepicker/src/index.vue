<!--
toolBar: Array [{text: '最近45天'，start: new Date(Date.now() - 3600 * 1000 * 24 * 45), end: new Date() }]
language: zh en
type: date daterange
width: date-picker width
format: 数据显示格式
v-model: 绑定数据
placeholder: tip
disabled: 是否可用
value: 初始值
icon: icon小图标
 -->

<template>
  <div
    class="datepicker"
    v-clickoutside="closePopup"
    :style="{'width': width}"
    >
    <YdInput
      readonly
      type="text"
      class="input"
      v-model="text"
      :placeholder="innerPlaceholder"
      ref="input"
      :disabled="disabled"
      @click="togglePopup"
      @mousedown="$event.preventDefault()">
    </YdInput>
    <i class="input-icon"
      :class="showCloseIcon ? 'input-icon__close' : 'input-icon__calendar'"
      v-html="icon"
      @click="clickIcon"></i>
    <div class="datepicker-popup"
         :class="[isDate() ? '' : 'range_date']"
         :style="position"
         ref="calendar"
         v-show="showPopup">
      <template v-if="isDate()">
        <Panel @select="showPopup = false" :type="type" v-model="currentValue" :show="showPopup"></Panel>
      </template>
      <template v-else>
        <div class="datepicker-top">
          <span v-for="item in ranges" @click="selectRange(item)">{{item.text}}</span>
        </div>
        <Panel :type="type" style="width:50%;box-shadow:1px 0 rgba(0, 0, 0, .1)"  v-model="currentValue[0]" :is-end="false" :end-at="currentValue[1]" :show="showPopup"></Panel>
        <Panel :type="type" style="width:50%;"  v-model="currentValue[1]" :is-end="true" :start-at="currentValue[0]" :show="showPopup"></Panel>
      </template>
    </div>
  </div>
</template>

<script>
import {formatDate, oneOf} from 'UI/utils/util'
import Languages from './language.js'
import Panel from './panel.vue'
import YdInput from '../../Input/index.vue'
export default {
  name: 'date-picker',
  props: {
    format: {
      type: String,
      default: '%Y-%m-%d'
    },
    width: {
      type: [String, Number],
      default: 250
    },
    placeholder: String,
    lang: {
      type: String,
      default: 'zh'
    },
    disabled: Boolean,
    type: {
      validator(val) {
        return oneOf(val, ['date', 'daterange'])
      },
      default: 'date'
    },
    icon: {
      type: String,
      default: ''
    },
    toolBar: Array,
    value: null

  },
  data () {
    return {
      showPopup: false,
      showCloseIcon: false,
      currentValue: this.value,
      position: null,
      ranges: []  // 快捷选项
    }
  },
  watch: {
    value: {
      handler (val) {
        if (oneOf(this.type, ['date'])) {
          this.currentValue = this.isValidDate(val) ? val : undefined
        } else {
          this.currentValue = this.isValidRange(val) ? val : [undefined, undefined]
        }
      },
      immediate: true
    },
    currentValue (val) {
      if ((this.isDate() && val) || (!this.isDate() && val[0] && val[1])) {
        this.$emit('input', val)
      }
    },
    showPopup (val) {
      if (val) this.$nextTick(this.displayPopup())
    }
  },
  computed: {
    translation () {
      return Languages[this.lang] || Languages['en'];
    },
    innerPlaceholder () {
      return this.placeholder || ( this.isDate() ? this.translation.placeholder.date : this.translation.placeholder.dateRange )
    },
    text () {
      if (this.isDate() && this.currentValue) {
        return this.stringify(this.currentValue)
      } else if (!this.isDate() && this.currentValue[0] && this.currentValue[1]) {
        return this.stringify(this.currentValue[0]) + ' ~ ' + this.stringify(this.currentValue[1])
      } else {
        return ''
      }
    }
  },
  methods: {
    isDate() {
      return oneOf(this.type, ['date'])
    },
    closePopup() {
      this.showPopup = false
    },
    togglePopup() {
      if(this.showPopup) {
        this.$refs.input.handleBlur()
        this.showPopup = false
      } else {
        this.$refs.input.autoFocus()
        this.showPopup = true
      }
    },
    clickIcon () {
      this.showCloseIcon ? this.$emit('input', '') : this.togglePopup()
    },
    stringify (date) {
      return formatDate(new Date(date), this.format)
    },
    isValidDate (date) {
      return !!new Date(date).getTime()
    },
    isValidRange (date) {
      return Array.isArray(date) &&
        date.length === 2 &&
        this.isValidDate(date[0]) &&
        this.isValidDate(date[1])
    },
    selectRange (range) {
      this.$emit('input', [range.start, range.end])
    },
    //
    initRanges () {
      this.ranges = this.toolBar && Array.isArray(this.toolBar) ? this.toolBar : []
    },
    displayPopup () {
      const dw = document.documentElement.clientWidth
      const dh = document.documentElement.clientHeight
      const InputRect = this.$el.getBoundingClientRect()
      const PopupRect = this.$refs.calendar.getBoundingClientRect()
      this.position = {}
      if (dw - InputRect.left < PopupRect.width && InputRect.right < PopupRect.width) {
        this.position.left = 1 - InputRect.left + 'px'
      } else if (InputRect.left + InputRect.width / 2 <= dw / 2) {
        this.position.left = 0
      } else {
        this.position.right = 0
      }
      if (InputRect.top <= PopupRect.height + 1 && dh - InputRect.bottom <= PopupRect.height + 1) {
        this.position.top = dh - InputRect.top - PopupRect.height - 1 + 'px'
      } else if (InputRect.top + InputRect.height / 2 <= dh / 2) {
        this.position.top = '100%'
      } else {
        this.position.bottom = '100%'
      }
    }
  },
  created () {
    this.initRanges()
  },
  directives: {
    clickoutside: {
      bind (el, binding, vnode) {
        el['@clickoutside'] = (e) => {
          if (!el.contains(e.target) && binding.expression && vnode.context[binding.expression]) {
            binding.value()
          }
        }
        document.addEventListener('click', el['@clickoutside'], true)
      },
      unbind (el) {
        document.removeEventListener('click', el['@clickoutside'], true)
      }
    }
  },
  components: {
    Panel, YdInput
  }
}
</script>


<style media="screen" lang="stylus">
.datepicker
  position relative
  display inline-block
  font 14px/1.5 "Helvetica Neue", Helvetica, Arial, "Microsoft Yahei", sans-serif
  .datepicker-popup
    position absolute
    width 250px
    margin-top 1px
    margin-bottom 1px
    border 1px solid #d9d9d9
    background-color #fff
    box-shadow 0 6px 12px rgba(0, 0, 0, .175)
    z-index 1000
  .range_date
    width 448px
  .input-icon
    top 0
    right 0
    position absolute
    width 15px
    height 100%
    color #888
    text-align center
    font-style normal
    background url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAA00lEQVQ4T72SzQ2CQBCF54UGKIES6EAswQq0BS/A3PQ0hAt0oKVQgiVYAkcuZMwSMOyCyRKNe9uf+d6b2Qf6csGtL8sy7vu+Zebn/E5EoiAIwjRNH/PzBUBEGiJqmPniAMw+YeZkFSAiJwA3j45aVT0wsxGitwOjDGDnASBVvU4OLQARRURk9e4CAcSqWn8CLHp3Ae6MXAe/B4yzUeMkz/P9ZgdFUQzFIwD/B4yKgwMTos0OtvzCHcDRJ0gAzlmW1VYSq6oKu66LfQBTjC2AT+Hamxcml5IRpPq3VQAAAABJRU5ErkJggg==')
    &:after
      content ''
      display inline-block
      width 0
      height 100%
      vertical-align middle
  .input-icon__calendar
    background-position center
    background-repeat no-repeat
  .input-icon__close
    &:before
      content '\2716'
      vertical-align middle
  .datepicker-top
    margin 0 12px
    line-height 34px
    border-bottom 1px solid rgba(0, 0, 0, .05)
  .datepicker-top
    span
      white-space nowrap
      cursor pointer
  .datepicker-top
    span
      &:hover
        color #ec5222
  .datepicker-top
    span
      &:after
        content ""
        margin 0 10px
        color #ec5222
</style>
