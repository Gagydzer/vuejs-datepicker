<template>
  <div :class="{'input-group' : bootstrapStyling}">
    <!-- Calendar Button -->
    <span v-if="calendarButton" class="vdp-datepicker__calendar-button" :class="{'input-group-prepend' : bootstrapStyling}" @click="showCalendar" v-bind:style="{'cursor:not-allowed;' : disabled}">
      <span :class="{'input-group-text' : bootstrapStyling}">
        <i :class="calendarButtonIcon">
          {{ calendarButtonIconContent }}
          <span v-if="!calendarButtonIcon">&hellip;</span>
        </i>
      </span>
    </span>
    <!-- Input -->
    <masked-input
      v-if="mask"
      :class="computedInputClass"
      :type="inline ? 'hidden' : 'text'"
      :name="name"
      :ref="refName"
      :id="id"
      :value="formattedValue"
      :mask="mask"
      :open-date="openDate"
      :clear-button="clearButton"
      :disabled="disabled"
      :required="required"
      @input="(value) => tempValue = value"
      @click.native="showCalendar"
      @keyup.native="parseTypedDate"
      @blur.native="inputBlurred"/>
    <input v-else
      :name="name"
      :ref="refName"
      :class="computedInputClass"
      :id="id"
      :value="formattedValue"
      :open-date="openDate"
      :clear-button="clearButton"
      :disabled="disabled"
      :required="required"
      :readonly="!typeable"
      @input="(value) => tempValue = value"
      @click="showCalendar"
      @keyup="parseTypedDate"
      @blur="inputBlurred"/>
    <!-- Clear Button -->
    <span v-if="clearButton && selectedDate" class="vdp-datepicker__clear-button" :class="{'input-group-append' : bootstrapStyling}" @click="clearDate()">
      <span :class="{'input-group-text' : bootstrapStyling}">
        <i :class="clearButtonIcon">
          <span v-if="!clearButtonIcon">&times;</span>
        </i>
      </span>
    </span>
    <slot name="afterDateInput"></slot>
  </div>
</template>
<script>
import maskedInput from 'vue-masked-input'
import { makeDateUtils, parseDate } from '../utils/DateUtils'

export default {
  props: {
    selectedDate: Date,
    resetTypedDate: [Date],
    format: [String, Function],
    translation: Object,
    inline: Boolean,
    id: String,
    name: String,
    refName: {
      type: String,
      default : 'input'
    },
    openDate: Date,
    placeholder: String,
    inputClass: [String, Object, Array],
    clearButton: Boolean,
    clearButtonIcon: String,
    calendarButton: Boolean,
    calendarButtonIcon: String,
    calendarButtonIconContent: String,
    disabled: Boolean,
    required: Boolean,
    typeable: Boolean,
    bootstrapStyling: Boolean,
    useUtc: Boolean
  },
  components: {
    maskedInput
  },
  data () {
    const constructedDateUtils = makeDateUtils(this.useUtc)
    return {
      tempValue: '',
      input: null,
      typedDate: false,
      utils: constructedDateUtils
    }
  },
  computed: {
    formattedValue () {
      if (!this.selectedDate) {
        return null
      }
      if (this.typedDate) {
        return this.typedDate
      }
      return typeof this.format === 'function'
        ? this.format(this.selectedDate)
        : this.utils.formatDate(new Date(this.selectedDate), this.format, this.translation)
    },
    mask () {
      // function not supported
      if (!this.format 
        || typeof this.format === 'function' || !/\./.test(this.format) ) {
      console.log('format', /\./.test(this.format), this.format)

          return false
        }
      if (this.format === true) return '11.11.1111'
      return this.format.replace(/[a-zA-Z]/g, '1').replace(/\./, '\.')
    },
    computedInputClass () {
      if (this.bootstrapStyling) {
        if (typeof this.inputClass === 'string') {
          return [this.inputClass, 'form-control'].join(' ')
        }
        return {'form-control': true, ...this.inputClass}
      }
      return this.inputClass
    }
  },
  watch: {
    resetTypedDate () {
      this.typedDate = false
      this.tempValue = false
    },
    selectedDate (val) {
      if (val) this.tempValue = this.utils.formatDate(new Date(this.selectedDate), this.format, this.translation)
    }
  },
  methods: {
    showCalendar () {
      this.$emit('showCalendar')
    },
    /**
     * Attempt to parse a typed date
     * @param {Event} event
     */
    parseTypedDate (event) {
      // close calendar if escape or enter are pressed
      if ([
        27, // escape
        13 // enter
      ].includes(event.keyCode)) {
        this.input.blur()
      }

      if (this.typeable && isNeedParse(this.tempValue, this.mask)) {
        const typedDate = this.parseDate(this.tempValue)
        if (typedDate) {
          // this.typedDate = this.input.value
          this.$emit('typedDate', new Date(typedDate))
        }
      }

      function isNeedParse (value, mask) {
        const maskOnlyNumbers = mask.replace(/\D/g, '')
        const onlyNumbers = value.replace(/\D/g, '')
        return maskOnlyNumbers.length === onlyNumbers.length
      }
    },
    /**
     * nullify the typed date to defer to regular formatting
     * called once the input is blurred
     */
    inputBlurred () {
      console.log('inputBlurred', this.parseDate(this.tempValue))
      if (this.typeable && !this.parseDate(this.tempValue)) {
        this.clearDate()
        this.tempValue = null
        this.typedDate = null
      }

      this.$emit('closeCalendar')
    },
    parseDate (value) {
      return parseDate(value, typeof this.format === 'function' ? undefined : this.format)
    },
    /**
     * emit a clearDate event
     */
    clearDate () {
      this.$emit('clearDate')
    }
  },
  mounted () {
    console.log('refs')
    this.input = !this.mask 
      ? this.$refs[this.refName]
      : this.$refs[this.refName].$refs.input
    if (!this.typeable) this.input.setAttribute('readonly', true)
  }
}
// eslint-disable-next-line
;
</script>
