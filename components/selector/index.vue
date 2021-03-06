<template>
  <div
    class="md-selector"
    :class="{
      'is-normal': !isCheck,
      'is-check': isCheck && isNeedConfirm
    }"
  >
    <md-popup
      v-model="isSelectorShow"
      position="bottom"
      :mask-closable="maskClosable"
      @show="$_onSelectorShow"
      @hide="$_onSelectorHide"
      @maskClick="$_onSelectorCancel"
    >
      <md-popup-title-bar
        :title="title"
        :ok-text="okText"
        :cancel-text="cancelText"
        @confirm="$_onSelectorConfirm"
        @cancel="$_onSelectorCancel"
      ></md-popup-title-bar>
      <md-scroll-view
        ref="scroll"
        class="md-selector-container"
        :scrolling-x="false"
        :style="{maxHeight: `${maxHeight}px`}"
      >
        <div class="md-selector-list">
          <md-radio
            ref="radio"
            :key="radioKey"
            :options="data"
            :default-index="defaultIndex"
            :invalid-index="invalidIndex"
            icon="circle-right"
            icon-inverse="circle"
            icon-size="md"
            is-across-border
            :optionRender="optionRender"
            :is-slot-scope="hasSlot"
            @change="$_onSelectorChoose"
          >
            <template slot-scope="{ option }">
              <slot :option="option"></slot>
            </template>
          </md-radio>
        </div>
      </md-scroll-view>
    </md-popup>
  </div>
</template>

<script>import Popup from '../popup'
import PopupTitlebar from '../popup/title-bar'
import Radio from '../radio'
import ScrollView from '../scroll-view'
import {noop} from '../_util'

export default {
  name: 'md-selector',

  components: {
    [Radio.name]: Radio,
    [Popup.name]: Popup,
    [PopupTitlebar.name]: PopupTitlebar,
    [ScrollView.name]: ScrollView,
  },

  props: {
    value: {
      type: Boolean,
      default: false,
    },
    data: {
      type: Array,
      default() {
        return []
      },
    },
    defaultIndex: {
      type: Number,
      default: -1,
    },
    invalidIndex: {
      type: [Array, Number],
      default() {
        return -1
      },
    },
    title: {
      type: String,
      default: '',
    },
    okText: {
      type: String,
      default: '',
    },
    cancelText: {
      type: String,
      default: '',
    },
    maskClosable: {
      type: Boolean,
      default: true,
    },
    isCheck: {
      type: Boolean,
      default: false,
    },
    optionRender: {
      type: Function,
      default: noop,
    },
    maxHeight: {
      type: Number,
      default: 400,
    },
  },

  data() {
    return {
      isSelectorShow: false,
      radioKey: Date.now(),
      activeIndex: -1,
      tmpActiveIndex: -1,
    }
  },

  computed: {
    isNeedConfirm() {
      return this.okText !== ''
    },
    hasSlot() {
      return !!this.$scopedSlots.default
    },
  },

  watch: {
    value(val) {
      this.isSelectorShow = val
    },
    isSelectorShow(val) {
      this.$emit('input', val)
    },
  },

  created() {
    this.value && (this.isSelectorShow = this.value)
    !this.isNeedConfirm && (this.activeIndex = this.defaultIndex)
    this.activeIndex = this.tmpActiveIndex = this.defaultIndex
  },

  methods: {
    // MARK: private methods
    $_getItemText(item) {
      const renderText = this.itemRender(item)
      return renderText || item.text || item.label
    },
    $_isActive(index) {
      const activeIndex = this.tmpActiveIndex
      if (activeIndex > -1) {
        return activeIndex === index
      } else {
        return this.defaultIndex === index
      }
    },
    $_isInvalid(index) {
      const invalidIndex = this.invalidIndex
      return Array.isArray(invalidIndex) ? !!~invalidIndex.indexOf(index) : index === invalidIndex
    },
    $_setScroller() {
      this.$refs.scroll.reflowScroller()
    },

    // MARK: events handler
    $_onSelectorConfirm() {
      if (this.tmpActiveIndex > -1) {
        this.activeIndex = this.tmpActiveIndex
        this.isSelectorShow = false
        this.$emit('confirm', this.data[this.activeIndex])
      }
    },
    $_onSelectorCancel() {
      this.isSelectorShow = false
      this.tmpActiveIndex = this.activeIndex

      if (this.tmpActiveIndex !== -1) {
        this.$refs.radio.selectByIndex(this.tmpActiveIndex)
      } else {
        // reset radio
        this.radioKey = Date.now()
      }

      this.$emit('cancel')
    },
    $_onSelectorChoose(item, index) {
      this.tmpActiveIndex = index
      if (!this.isNeedConfirm) {
        this.activeIndex = index
        this.isSelectorShow = false
      }

      this.$emit('choose', item)
    },
    $_onSelectorShow() {
      /* istanbul ignore next  */
      this.$_setScroller()
      this.$emit('show')
    },
    $_onSelectorHide() {
      /* istanbul ignore next  */
      this.$emit('hide')
    },
  },
}
</script>

<style lang="stylus">
.md-selector
  .md-popup
    z-index selector-zindex !important
    .md-popup-box
      background-color color-bg-base
    .md-selector-container
      background color-bg-base
      overflow hidden
  .md-field-item.selected
    color selector-active-color !important
  &.is-check
    .md-field-item.selected .md-icon
      fill selector-active-color !important
      .md-field-item-content.left
        padding-right 32px
  &.is-normal
    .md-field-item-content.left
      justify-content center
    .md-field-item .md-icon
      display none
</style>
