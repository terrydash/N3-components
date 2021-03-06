<template>
  <div class="{{prefixCls}}-datepicker {{prefixCls}}-timepicker {{prefixCls}}-datetimepicker" v-el:datetimepicker>
    <n3-input
      :width="width"
      :name="name" 
      :rules="rules" 
      :validate="validate" 
      :has-feedback="hasFeedback"
      :placeholder="placeholder"
      :custom-validate="customValidate"
      :readonly="true"
      :disabled="disabled"
      @click="inputClick"
      :value.sync="value">
    </n3-input>
      <div 
        class="{{prefixCls}}-datepicker-popup" 
        v-show="displayDayView" 
        :style="{width:popWidth}" 
        transition="fadeDown">
          <div class="{{prefixCls}}-datepicker-inner">
              <div class="{{prefixCls}}-datepicker-body">
                  <div class="{{prefixCls}}-datepicker-ctrl">
                      <span 
                        class="{{prefixCls}}-month-btn {{prefixCls}}-datepicker-preBtn" 
                        @click="preNextMonthClick(0)">&lt;</span>
                      <span 
                        class="{{prefixCls}}-month-btn {{prefixCls}}-datepicker-nextBtn" 
                        @click="preNextMonthClick(1)">&gt;</span>
                      <p @click="switchMouthView">
                      {{stringifyDayHeader(currDate)}}
                      </p>
                  </div>
                  <div class="{{prefixCls}}-datepicker-weekRange">
                      <span v-for="w in weekRange">{{w}}</span>
                  </div>
                  <div class="{{prefixCls}}-datepicker-dateRange">
                      <span v-for="d in dateRange" 
                      :class="[d.sclass,prefixCls + '-datetimepicker-date-span']" 
                      @click="daySelect(d.date,d.sclass)">{{d.text}}</span>
                  </div>
              </div>
          </div>
          <div class="{{prefixCls}}-timepicker-con">
            <div class="{{prefixCls}}-timepicker-slider-sin-wrap" v-if="hour" data-role="hour">
              <n3-slider 
                class="{{prefixCls}}-timepicker-slider"
                :value.sync="time.hour" 
                orientation="vertical" 
                :max="hourRange[1]" 
                :min="hourRange[0]" >
              </n3-slider>
            </div>
            <div class="{{prefixCls}}-timepicker-slider-sin-wrap" v-if="minute" data-role="minute">
              <n3-slider
                class="{{prefixCls}}-timepicker-slider" 
                :value.sync="time.minute" 
                orientation="vertical" 
                :max="minuteRange[1]" 
                :min="minuteRange[0]" >
              </n3-slider>
            </div>
            <div class="{{prefixCls}}-timepicker-slider-sin-wrap" v-if="second" data-role="second">
              <n3-slider 
                class="{{prefixCls}}-timepicker-slider"
                :value.sync="time.second" 
                orientation="vertical" 
                :max="secondRange[1]" 
                :min="secondRange[0]" >
              </n3-slider>
            </div>
          </div>
      </div>
      <div class="{{prefixCls}}-datepicker-popup" v-show="displayMouthView">
        <div class="{{prefixCls}}-datepicker-inner">
            <div class="{{prefixCls}}-datepicker-body">
                <div class="{{prefixCls}}-datepicker-ctrl">
                    <span 
                      class="{{prefixCls}}-month-btn {{prefixCls}}-datepicker-preBtn" 
                      @click="preNextYearClick(0)">&lt;</span>
                    <span 
                      class="{{prefixCls}}-month-btn {{prefixCls}}-datepicker-nextBtn" 
                      @click="preNextYearClick(1)">&gt;</span>
                    <p @click="switchDecadeView">
                    {{stringifyYearHeader(currDate)}}
                    </p>
                </div>
                <div class="{{prefixCls}}-datepicker-mouthRange">
                	<template v-for="m in mouthNames">
	                    <span 
                        :class="monthClassobj(m)"
                        @click="mouthSelect($index)">
	                      {{m.substr(0,3)}}
	                    </span>
                    </template>
                </div>
            </div>
        </div>
      </div>
      <div class="{{prefixCls}}-datepicker-popup" v-show="displayYearView">
        <div class="{{prefixCls}}-datepicker-inner">
            <div class="{{prefixCls}}-datepicker-body">
                <div class="{{prefixCls}}-datepicker-ctrl">
                    <span 
                      class="{{prefixCls}}-month-btn {{prefixCls}}-datepicker-preBtn"
                      @click="preNextDecadeClick(0)">&lt;</span>
                    <span 
                      class="{{prefixCls}}-month-btn {{prefixCls}}-datepicker-nextBtn" 
                      @click="preNextDecadeClick(1)">&gt;</span>
                    <p>
                    {{stringifyDecadeHeader(currDate)}}
                    </p>
                </div>
                <div class="{{prefixCls}}-datepicker-mouthRange {{prefixCls}}-datepicker-decadeRange">
                	<template v-for="decade in decadeRange">
                		<span
                      :class="yearClassobj(decade)"
	                    @click.stop="yearSelect(decade.text)">
	                      {{decade.text}}
	                  </span>
					        </template>
                </div>
            </div>
        </div>
      </div>
</div>
</template>

<script>
import n3Slider from './n3Slider'
import n3Input from './n3Input'
import inputMixin from './inputMixin'
import EventListener from './utils/EventListener'
import type from './utils/type'

export default {
  mixins: [inputMixin],
  props: {
    value: {
      type: String,
      twoWay: true
    },
    format: {
      default: 'yyyy-MM-dd hh:mm:ss'
    },
    disabledDaysOfWeek: {
      type: Array,
      default () {
        return []
      }
    },
    hourRange: {
      type: Array,
      default () {
        return [0, 23]
      }
    },
    minuteRange: {
      type: Array,
      default () {
        return [0, 59]
      }
    },
    secondRange: {
      type: Array,
      default () {
        return [0, 59]
      }
    },
    onHide: {
      type: Function
    },
    prefixCls: {
      type: String,
      default: 'n3'
    }
  },
  data () {
    return {
      weekRange: ['日', '一', '二', '三', '四', '五', '六'],
      dateRange: [],
      decadeRange: [],
      currDate: new Date(),
      displayDayView: false,
      displayMouthView: false,
      displayYearView: false,
      time: {
        hour: 0,
        minute: 0,
        second: 0
      },
      date: '',
      mouthNames: [
        '一月', '二月', '三月',
        '四月', '五月', '六月',
        '七月', '八月', '九月',
        '十月', '十一月', '十二月'
      ]
    }
  },
  watch: {
    displayDayView () {
      this.dispatchHide()
    },
    displayMouthView () {
      this.dispatchHide()
    },
    displayYearView () {
      this.dispatchHide()
    },
    currDate () {
      this.getDateRange()
    },
    date () {
      this.value = this.date + ' ' + this.handTime()
    },
    time: {
      deep: true,
      handler (val) {
        this.value = this.date + ' ' + this.handTime()
      }
    }
  },
  computed: {
    popWidth () {
      let width = 293
      this.hour ? width += 44 : 0
      this.minute ? width += 44 : 0
      this.second ? width += 44 : 0

      return width + 'px'
    },
    hour () {
      if (this.format.indexOf('hh') > -1) {
        return true
      } else {
        return false
      }
    },
    minute () {
      if (this.format.indexOf('mm') > -1) {
        return true
      } else {
        return false
      }
    },
    second () {
      if (this.format.indexOf('ss') > -1) {
        return true
      } else {
        return false
      }
    }
  },
  methods: {
    monthClassobj (m) {
      let {prefixCls, date, mouthNames, parse, currDate} = this
      let klass = {}

      klass[prefixCls + '-datepicker-dateRange-item-active'] =
      date && parse(date) && mouthNames[parse(date).getMonth()] === m && currDate.getFullYear() === parse(date).getFullYear()

      return klass
    },
    yearClassobj (decade) {
      let {prefixCls, date, parse} = this
      let klass = {}

      klass[prefixCls + '-datepicker-dateRange-item-active'] =
      date && parse(date) && parse(date).getFullYear() === decade.text

      return klass
    },
    dispatchHide () {
      var show = this.displayDayView || this.displayMouthView || this.displayYearView
      if (!show) {
        if (type.isFunction(this.onHide)) {
          this.onHide()
        }
      }
    },
    close () {
      this.displayDayView = this.displayMouthView = this.displayMouthView = false
    },
    handTime () {
      var ret = ''
      this.time.hour > this.hourRange[1] ? this.time.hour = this.hourRange[1] : 0
      this.time.minute > this.minuteRange[1] ? this.time.minute = this.minuteRange[1] : 0
      this.time.second > this.secondRange[1] ? this.time.second = this.secondRange[1] : 0
      this.time.hour < this.hourRange[0] ? this.time.hour = this.hourRange[0] : 0
      this.time.minute < this.minuteRange[0] ? this.time.minute = this.minuteRange[0] : 0
      this.time.second < this.secondRange[0] ? this.time.second = this.secondRange[0] : 0

      if (this.hour) {
        ret += this.fix(this.time.hour, 2) + ':'
      }

      if (this.minute) {
        ret += this.fix(this.time.minute, 2) + ':'
      }

      if (this.second) {
        ret += this.fix(this.time.second, 2) + ':'
      }

      return ret.substr(0, ret.length - 1)
    },
    fix (num, length) {
      num = parseInt(num)
      num = isNaN(num) ? 0 : num
      return ('' + num).length < length ? ((new Array(length + 1)).join('0') + num).slice(-length) : '' + num
    },
    inputClick () {
      if (this.disabled) return
      if (this.displayMouthView || this.displayYearView) {
        this.displayDayView = false
      } else {
        this.displayDayView = !this.displayDayView
      }
    },
    preNextDecadeClick (flag) {
      const year = this.currDate.getFullYear()
      const mouths = this.currDate.getMonth()
      const date = this.currDate.getDate()

      if (flag === 0) {
        this.currDate = new Date(year - 10, mouths, date)
      } else {
        this.currDate = new Date(year + 10, mouths, date)
      }
    },
    preNextMonthClick (flag) {
      const year = this.currDate.getFullYear()
      const month = this.currDate.getMonth()
      const date = this.currDate.getDate()

      if (flag === 0) {
        const preMonth = this.getYearMonth(year, month - 1)
        this.currDate = new Date(preMonth.year, preMonth.month, date)
      } else {
        const nextMonth = this.getYearMonth(year, month + 1)
        this.currDate = new Date(nextMonth.year, nextMonth.month, date)
      }
    },
    preNextYearClick (flag) {
      const year = this.currDate.getFullYear()
      const mouths = this.currDate.getMonth()
      const date = this.currDate.getDate()

      if (flag === 0) {
        this.currDate = new Date(year - 1, mouths, date)
      } else {
        this.currDate = new Date(year + 1, mouths, date)
      }
    },
    yearSelect (year) {
      this.displayYearView = false
      this.displayMouthView = true
      this.currDate = new Date(year, this.currDate.getMonth(), this.currDate.getDate())
    },
    daySelect (date, klass) {
      if (klass.indexOf(this.prefixCls + '-datepicker-item-disable') > -1) {
        return false
      } else {
        this.currDate = date
        this.date = this.stringify(this.currDate)
      }
    },
    switchMouthView () {
      this.displayDayView = false
      this.displayMouthView = true
    },
    switchDecadeView () {
      this.displayMouthView = false
      this.displayYearView = true
    },
    mouthSelect (index) {
      this.displayMouthView = false
      this.displayDayView = true
      this.currDate = new Date(this.currDate.getFullYear(), index, this.currDate.getDate())
    },
    getYearMonth (year, month) {
      if (month > 11) {
        year++
        month = 0
      } else if (month < 0) {
        year--
        month = 11
      }
      return {year: year, month: month}
    },
    stringifyDecadeHeader (date) {
      const yearStr = date.getFullYear().toString()
      const firstYearOfDecade = yearStr.substring(0, yearStr.length - 1) + 0
      const lastYearOfDecade = parseInt(firstYearOfDecade, 10) + 10
      return firstYearOfDecade + '-' + lastYearOfDecade
    },
    stringifyDayHeader (date) {
      return this.mouthNames[date.getMonth()] + ' ' + date.getFullYear()
    },
    parseMouth (date) {
      return this.mouthNames[date.getMonth()]
    },
    stringifyYearHeader (date) {
      return date.getFullYear()
    },
    stringifyTime (date) {
      const hour = date.getHours()
      const minute = date.getMinutes()
      const second = date.getSeconds()

      return {
        hour: hour,
        minute: minute,
        second: second
      }
    },
    stringify (date, format = this.format) {
      if (isNaN(date.getFullYear())) return ''
      const dateFormat = format.split(/\s+/)[0]
      const year = date.getFullYear()
      const month = date.getMonth() + 1
      const day = date.getDate()

      return dateFormat
        .replace(/yyyy/g, year)
        .replace(/MMMM/g, month)
        .replace(/MMM/g, month)
        .replace(/MM/g, ('0' + month).slice(-2))
        .replace(/dd/g, ('0' + day).slice(-2))
        .replace(/yy/g, year)
        .replace(/M(?!a)/g, month)
        .replace(/d/g, day)
    },
    parse (str) {
      const date = new Date(str)
      return isNaN(date.getFullYear()) ? null : date
    },
    getDayCount (year, month) {
      const dict = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

      if (month === 1) {
        if ((year % 400 === 0) || (year % 4 === 0 && year % 100 !== 0)) {
          return 29
        }
        return 28
      }

      return dict[month]
    },
    getDateRange () {
      this.dateRange = []
      this.decadeRange = []
      const time = {
        year: this.currDate.getFullYear(),
        month: this.currDate.getMonth(),
        day: this.currDate.getDate()
      }
      const yearStr = time.year.toString()
      const firstYearOfDecade = (yearStr.substring(0, yearStr.length - 1) + 0) - 1
      for (let i = 0; i < 12; i++) {
        this.decadeRange.push({
          text: firstYearOfDecade + i
        })
      }

      const currMonthFirstDay = new Date(time.year, time.month, 1)
      let firstDayWeek = currMonthFirstDay.getDay() + 1
      if (firstDayWeek === 0) {
        firstDayWeek = 7
      }
      const dayCount = this.getDayCount(time.year, time.month)
      if (firstDayWeek > 1) {
        const preMonth = this.getYearMonth(time.year, time.month - 1)
        const prevMonthDayCount = this.getDayCount(preMonth.year, preMonth.month)
        for (let i = 1; i < firstDayWeek; i++) {
          const dayText = prevMonthDayCount - firstDayWeek + i + 1
          this.dateRange.push({
            text: dayText,
            date: new Date(preMonth.year, preMonth.month, dayText),
            sclass: this.prefixCls + '-datepicker-item-gray'
          })
        }
      }

      for (let i = 1; i <= dayCount; i++) {
        const date = new Date(time.year, time.month, i)
        const week = date.getDay()
        let sclass = ''
        this.disabledDaysOfWeek.forEach((el) => {
          if (week === parseInt(el, 10)) sclass = this.prefixCls + '-datepicker-item-disable'
        })

        if (i === time.day) {
          if (this.date) {
            const valueDate = this.parse(this.date)
            if (valueDate) {
              if (valueDate.getFullYear() === time.year && valueDate.getMonth() === time.month) {
                sclass = this.prefixCls + '-datepicker-dateRange-item-active'
              }
            }
          }
        }
        this.dateRange.push({
          text: i,
          date: date,
          sclass: sclass
        })
      }
      if (this.dateRange.length < 42) {
        const nextMonthNeed = 42 - this.dateRange.length
        const nextMonth = this.getYearMonth(time.year, time.month + 1)

        for (let i = 1; i <= nextMonthNeed; i++) {
          this.dateRange.push({
            text: i,
            date: new Date(nextMonth.year, nextMonth.month, i),
            sclass: this.prefixCls + '-datepicker-item-gray'
          })
        }
      }
    }
  },
  created () {
    if (this.value) {
      this.date = this.stringify(new Date(this.value))
      this.time = this.stringifyTime(new Date(this.value))
    }
  },
  ready () {
    var self = this
    this.currDate = this.parse(this.date) || this.parse(new Date())
    this._closeEvent = EventListener.listen(window, 'click', (e) => {
      if (!self.displayDayView && !self.displayMouthView && !self.displayMouthView) {
        return
      }
      var check = e.target.className.indexOf(this.prefixCls + '-datetimepicker-date-span') > -1
      if (!self.$el.contains(e.target) && !check) self.close()
    })
  },
  beforeDestroy () {
    if (this._closeEvent) this._closeEvent.remove()
  },
  components: {
    n3Slider,
    n3Input
  },
  filters: {
    fix: {
      read (val) {
        return this.fix(val, 2)
      },
      write (val) {
        return parseInt(val)
      }
    }
  }
}
</script>