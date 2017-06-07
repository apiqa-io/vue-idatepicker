<template>
  <div ref="datepicker" class="datepicker" :style="{ width: mergedOptions.styles.pickerWidth, boxShadow: mergedOptions.styles.pickerShadow }" v-if="opened">
    <div class="picker-helper">
      <span :style="{color: !day ? mergedOptions.styles.placeholderColor : null}">{{ day ? day : mergedOptions.emptyDay }}</span>
      <span :style="{color: !month ? mergedOptions.styles.placeholderColor : null}">{{ month ? mergedOptions.months[month - 1] : mergedOptions.emptyMonth }}</span>
      <span :style="{color: !year ? mergedOptions.styles.placeholderColor : null}">{{ year ? year : mergedOptions.emptyYear }}</span>
      <a :style="{color: mergedOptions.styles.clearColor}" @click="clearDate">{{ mergedOptions.clear }}</a>
    </div>
    <div class="picker">
      <div class="part day" @mousewheel="removeTransfrom">
        <ul ref="days">
          <li v-for="d in 31" :style="getListStyles(d, 'day')" :class="{ disabled: isDisabledDay(d), selected: isSelected(d, 'day') }" @click="setDay(d, true)">{{ d }}</li>
        </ul>
      </div>
      <div class="part month" @mousewheel="removeTransfrom">
        <ul ref="months">
          <li v-for="m in 12" :style="getListStyles(m, 'month')" :class="{ disabled: isDisabledMonth(m), selected: isSelected(m, 'month') }" @click="setMonth(m, true)">{{ mergedOptions.months[m - 1] }}</li>
        </ul>
      </div>
      <div class="part year" @mousewheel="removeTransfrom">
        <ul ref="years">
          <li v-for="y in years" :style="getListStyles(y, 'year')" @click="setYear(y, true)" :class="{ selected: isSelected(y, 'year') }">{{ y }}</li>
        </ul>
      </div>
    </div>
  </div>
  <div ref="datepicker" class="datepicker-info" @click="activate" v-else>
    <span class="date" :class="{ required: !day && !month && !year && required }">
      <span :style="{color: !day ? mergedOptions.styles.placeholderColor : null}">{{ day ? day : mergedOptions.emptyDay }}</span>
      <span :style="{color: !month ? mergedOptions.styles.placeholderColor : null}">{{ month ? mergedOptions.months[month - 1] : mergedOptions.emptyMonth }}</span>
      <span :style="{color: !year ? mergedOptions.styles.placeholderColor : null}">{{ year ? year : mergedOptions.emptyYear }}</span>
    </span>
    <slot></slot>
  </div>
</template>

<script>
  export default {
    name: 'Datepicker',
    props: {
      required: {
        type: Boolean,
        default: false
      },
      readonly: { type: Boolean },
      options: {
        type: Object,
        default () {
          return {}
        }
      },
      date: null,
      action: { type: Function }
    },
    data () {
      return {
        initialDate: null,
        day: null,
        month: null,
        year: null,
        opened: false,
        monthDays: { 1: 31, 2: 28, 3: 31, 4: 30, 5: 31, 6: 30, 7: 31, 8: 31, 9: 30, 10: 31, 11: 30, 12: 31 },
        now: new Date(),
        mergedOptions: {},
        defaultOptions: {
          emptyDay: 'day',
          emptyMonth: 'month',
          emptyYear: 'year',
          clear: 'clear',
          months: ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
          startDate: new Date(1897, 0, 1),
          finishDate: new Date(),
          styles: {
            pickerWidth: '100%',
            pickerShadow: '3px 4px 18px 0px rgba(115, 115, 115, 0.6)',
            clearColor: '#358ed7',
            selectedBackground: '#358ed7',
            selectedColor: '#fff',
            disabledColor: 'rgba(115, 115, 115, 0.7)',
            placeholderColor: 'rgba(115, 115, 115, 0.7)',
            textColor: '#1b212b'
          }
        }
      }
    },
    watch: {
      date () {
        this.setInitialDate(this.date)
      },
      options () {
        this.mergedOptions = {...this.defaultOptions, ...this.options}
      }
    },
    computed: {
      age () {
        return this.year && this.month && this.day ? new Date(this.year, this.month - 1, this.day) : null
      },
      years () {
        const startYear = this.mergedOptions.startDate.getFullYear()
        const finishYear = this.mergedOptions.finishDate.getFullYear()
        return Array(finishYear - startYear + 1).fill().map((item, idx) => startYear + idx)
      }
    },
    methods: {
      getListStyles (value, type) {
        let style = {}

        style.color = (type === 'day' && this.isDisabledDay(value)) || (type === 'month' && this.isDisabledMonth(value))
          ? this.mergedOptions.styles.disabledColor
          : this.mergedOptions.styles.textColor

        if (this.isSelected(value, type)) {
          style.color = this.mergedOptions.styles.selectedColor
          style.backgroundColor = this.mergedOptions.styles.selectedBackground
        }

        return style
      },
      isDisabledMonth (month) {
        const startYear = this.mergedOptions.startDate.getFullYear()
        const startMonth = this.mergedOptions.startDate.getMonth() + 1
        const finishYear = this.mergedOptions.finishDate.getFullYear()
        const finishMonth = this.mergedOptions.finishDate.getMonth() + 1
        return this.year
          ? (this.year === startYear && month < startMonth) || (this.year === finishYear && month > finishMonth)
          : false
      },
      isDisabledDay (day) {
        let disabled = false
        // check month days
        if (this.month && day > this.monthDays[this.month]) disabled = true
        // check start day
        const startYear = this.mergedOptions.startDate.getFullYear()
        const startMonth = this.mergedOptions.startDate.getMonth() + 1
        const startDay = this.mergedOptions.startDate.getDate()
        if (this.year && this.year === startYear && this.month && this.month >= startMonth && day > startDay) disabled = true
        // check finish day
        const finishYear = this.mergedOptions.finishDate.getFullYear()
        const finishMonth = this.mergedOptions.finishDate.getMonth() + 1
        const finishDay = this.mergedOptions.finishDate.getDate()
        if (this.year && this.year === finishYear && this.month && this.month >= finishMonth && day > finishDay) disabled = true
        return disabled
      },
      isSelected (value, type) {
        return value === this[type]
      },
      setYear (year, useAnimation) {
        this.year = year
        this.$nextTick(() => {
          this.scrollList('years', useAnimation)
        })
        // check if it is a leap-year
        this.monthDays[2] = ((year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0)) ? 29 : 28
        // check if chosen invalid month and day and set closest valid
        if (this.month && this.isDisabledMonth(this.month)) {
          let direction = (this.year === this.mergedOptions.startDate.getFullYear()) && (this.month < (this.mergedOptions.startDate.getMonth() + 1))
          this.searchAndSetClosest('month', direction)
        }
        if (this.day && this.isDisabledDay(this.day)) {
          let direction = (this.year === this.mergedOptions.startDate.getFullYear()) &&
                          (this.month === (this.mergedOptions.startDate.getMonth() + 1)) &&
                          (this.day < this.mergedOptions.startDate.getDate())
          this.searchAndSetClosest('day', direction)
        }
      },
      setMonth (month, useAnimation) {
        if (this.isDisabledMonth(month)) return
        this.month = month
        this.$nextTick(() => {
          this.scrollList('months', useAnimation)
        })
        // check if chosen invalid day and set closest valid
        if (this.day && this.isDisabledDay(this.day)) {
          let direction = (this.year === this.mergedOptions.startDate.getFullYear()) &&
                          (this.month === (this.mergedOptions.startDate.getMonth() + 1)) &&
                          (this.day < this.mergedOptions.startDate.getDate())
          this.searchAndSetClosest('day', direction)
        }
      },
      setDay (day, useAnimation) {
        if (this.isDisabledDay(day)) return
        this.day = day
        this.$nextTick(() => {
          this.scrollList('days', useAnimation)
        })
      },
      // helper function to prevent code dublication
      searchAndSetClosest (type, increaseSearching = false) {
        const capitalizedType = type.replace(/\b\w/g, l => l.toUpperCase())
        let disabledFunc = 'isDisabled' + capitalizedType
        let setFunc = 'set' + capitalizedType
        let possibleValue = this[type]
        if (increaseSearching) {
          while (this[disabledFunc](++possibleValue)) { }
        } else {
          while (this[disabledFunc](--possibleValue)) { }
        }
        this[setFunc](possibleValue)
      },
      // removes transform to start scrolling again
      removeTransfrom ($event) {
        let ul = $event.target.tagName === 'LI' ? $event.target.parentNode : $event.target.firstChild
        ul.style.transform = 'none'
      },
      // scroll item to 6th position if possible and set transform if not
      scrollList (list, useAnimation) {
        const li = this.$refs[list].querySelector('.selected')
        const ul = this.$refs[list]
        let scrollTop = li.offsetTop - 90
        const scrollMax = ul.scrollHeight + 10 - 234
        // hack of 1px miss
        if (list === 'months') {
          const idx = Array.from(ul.children).findIndex((el) => {
            return el.className === 'selected'
          })
          if (idx >= 5) scrollTop--
        }
        if (scrollTop > scrollMax) {
          const unscrollable = scrollMax - scrollTop
          if (useAnimation) {
            this.scrollTo(ul, scrollMax, 200)
            ul.style.transition = 'transform .2s ease-in'
          } else {
            ul.scrollTop = scrollMax
            ul.style.transition = 'none'
          }
          ul.style.transform = `translateY(${unscrollable}px)`
        } else if (scrollTop < 0) {
          if (useAnimation) {
            this.scrollTo(ul, 0, 200)
            ul.style.transition = 'transform .2s ease-in'
          } else {
            ul.scrollTop = 0
            ul.style.transition = 'none'
          }
          ul.style.transform = `translateY(${-scrollTop}px)`
        } else {
          if (useAnimation) {
            this.scrollTo(ul, scrollTop, 200)
            ul.style.transition = 'transform .2s ease-in'
          } else {
            ul.scrollTop = scrollTop
            ul.style.transition = 'none'
          }
          ul.style.transform = 'none'
        }
      },
      // smooth scrolling
      scrollTo (element, to, duration) {
        if (duration <= 0) return
        const difference = to - element.scrollTop
        const perTick = difference / duration * 10

        setTimeout(() => {
          element.scrollTop = element.scrollTop + perTick
          if (element.scrollTop === to) return
          this.scrollTo(element, to, duration - 10)
        }, 10)
      },
      activate () {
        if (!this.readonly) {
          this.opened = true
          if (this.initialDate !== null) {
            this.setYear(this.initialDate.getFullYear(), false)
            this.setMonth(this.initialDate.getMonth() + 1, false)
            this.setDay(this.initialDate.getDate(), false)
          } else {
            this.$nextTick(() => {
              const ul = this.$refs.years
              const scrollTop = ul.scrollHeight / 2
              ul.scrollTop = scrollTop
            })
          }

          setTimeout(() => {
            document.addEventListener('click', this.deactivate)
          }, 0)
        }
      },
      deactivate (e) {
        if (this.$refs.datepicker && !this.$refs.datepicker.contains(e.target)) {
          this.opened = false
          let error
          let result
          if (this.year && this.month && this.day) {
            result = this.age
            error = null
          } else {
            result = null
            error = {
              day: !this.day,
              month: !this.month,
              year: !this.year
            }
          }
          this.$emit('datechange', error, result)
          document.removeEventListener('click', this.deactivate)
        }
      },
      setInitialDate (date) {
        this.initialDate = date ? new Date(date * 1000) : new Date()
        this.year = this.initialDate.getFullYear()
        this.month = this.initialDate.getMonth() + 1
        this.day = this.initialDate.getDate()
      },
      clearDate () {
        this.year = null
        this.month = null
        this.day = null
      }
    },
    created () {
      this.mergedOptions = {...this.defaultOptions, ...this.options}
      if (this.date) {
        this.setInitialDate(this.date)
      }
    }
  }
</script>

<style lang="css" scoped>
  ul {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: -17px;
    overflow-y: scroll;
    padding: 0;
    margin: 5px 0;
  }

  li {
    list-style-type: none;
    width: 100%;
    cursor: pointer;
    padding: 2px 15px;
    padding-right: 0;
    height: 18px;
  }

  li.disabled {
    cursor: default;
  }

  li:not(.disabled):not(.selected):hover {
    background-color: #E9F0F2;
  }

  .datepicker {
    position: absolute;
    left: 0;
    font-size: 12px;
    border-radius: 5px;
    z-index: 1;
    font-family: Arial;
  }

  .picker {
    display: flex;
    align-items: stretch;
    height: calc(18px * 13);
    width: 100%;
    background-color: #fff;
    border-top: 1px solid rgba(115, 115, 115, 0.3);
    border-bottom-left-radius: 5px;
    border-bottom-right-radius: 5px;
  }

  .part {
    height: 100%;
    width: 100%;
    position: relative;
    overflow: hidden;
    border-right: 1px solid rgba(115, 115, 115, 0.3);
  }

  .part.year {
    border-right: none;
  }

  .picker-helper {
    position: relative;
    border-top-left-radius: 5px;
    border-top-right-radius: 5px;
    width: 100%;
    height: auto;
    padding: 5px 15px;
    background-color: #fff;
  }

  .picker-helper span {
    text-transform: lowercase;
  }

  .picker-helper a {
    float: right;
    text-transform: uppercase;
    letter-spacing: 1px;
    font-size: 10px;
    line-height: 15px;
    letter-spacing: .7px;
    cursor: pointer;
  }

  .datepicker-info {
    display: inline-flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
    position: relative;
    font-family: Arial;
  }

  .date {
    border-bottom: 1px dashed rgba(0, 0, 0, 0.25);
    white-space: nowrap;
  }

  .date.required::after {
    content: ' *';
    color: red;
  }

  .date span {
    text-transform: lowercase;
  }
</style>
