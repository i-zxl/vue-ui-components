<template>
  <div class="calendar">
    <div class="calendar-header">
      <a class="calendar__prev-icon" @click="changeYear(-1)">&laquo;</a>
      <a v-show="currentPanel === 'date'" class="calendar__prev-icon" @click="changeMonth(-1)">&lsaquo;</a>
      <a class="calendar__next-icon" @click="changeYear(1)">&raquo;</a>
      <a v-show="currentPanel === 'date'" class="calendar__next-icon" @click="changeMonth(1)" >&rsaquo;</a>
      <a @click="showMonths">{{months[currentMonth]}}</a>
      <a @click="showYears">{{currentYear}}</a>
    </div>
    <div class="calendar-content">
      <table class="calendar-table" v-show="currentPanel === 'date'">
        <thead>
          <tr>
            <th v-for="day in days">{{day}}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in dates">
            <td v-for="cell in row"
                :title="cell.title"
                :class="getClasses(cell)"
                @click="selectDate(cell)">{{cell.day}}</td>
          </tr>
        </tbody>
      </table>
      <div class="calendar-year" v-show="currentPanel === 'years'">
        <a v-for="year in years" @click="selectYear(year)" :class="{'current': currentYear === year}">{{year}}</a>
      </div>
      <div class="calendar-month" v-show="currentPanel === 'months'">
        <a v-for="(month, index) in months" @click="selectMonth(index)" :class="{'current': currentMonth === index}">{{month}}</a>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    startAt: null,
    endAt: null,
    value: null,
    show: Boolean,
    isEnd: Boolean
  },
  data () {
    const translation = this.$parent.translation
    return {
      days: translation.days,
      months: translation.months,
      dates: [],
      now: new Date(), // calendar-header 显示的时间, 用于切换日历
      years: [], // 年代面板
      currentPanel: 'date'
    }
  },
  computed: {
    currentYear () {
      return this.now.getFullYear()
    },
    currentMonth () {
      return this.now.getMonth()
    }
  },
  created () {
    this.updateCalendar()
  },
  watch: {
    show (val) {
      if (val) {
        this.currentPanel = 'date'
        this.updateNow()
      }
    },
    value: {
      handler: 'updateNow',
      immediate: true
    },
    now: 'updateCalendar',
  },
  methods: {
    updateNow () {
      let now = '';
      if (this.value) {
        now = new Date(this.value);
      } else if (!this.isEnd) {
        now = new Date();
      } else {
        now = new Date();
        let month = now.getMonth()+1,
            year = now.getFullYear();
        if (month>11) {
          let year = new Date().getFullYear()+1
          mouth = 0
        }
        now.setFullYear(year);
        now.setMonth(month);
      }
      now.setDate(1)
      this.now = now;
    },
    // 更新面板选择时间
    updateCalendar () {
      function getCalendar (time, firstday, length, classes) {
        return Array.apply(null, { length }).map((v, i) => { // eslint-disable-line
          let day = firstday + i
          const date = new Date(time.getFullYear(), time.getMonth(), day)
          return {
            title: date.toLocaleDateString(),
            date,
            day,
            classes
          }
        })
      }
      const time = new Date(this.now)
      time.setDate(0) // 把时间切换到上个月最后一天
      const lastMonthLength = time.getDay() + 1  // time.getDay() 0是星期天, 1是星期一 ...
      const lastMonthfirst = time.getDate() - (lastMonthLength - 1)
      const lastMonth = getCalendar(time, lastMonthfirst, lastMonthLength, 'lastMonth')

      time.setMonth(time.getMonth() + 2, 0) // 切换到这个月最后一天
      const curMonthLength = time.getDate()
      const curMonth = getCalendar(time, 1, curMonthLength, 'curMonth')

      time.setMonth(time.getMonth() + 1, 1)
      const nextMonthLength = 42 - (lastMonthLength + curMonthLength)
      const nextMonth = getCalendar(time, 1, nextMonthLength, 'nextMonth')

      // 分割数组
      let index = 0
      let resIndex = 0
      const arr = lastMonth.concat(curMonth, nextMonth)
      const result = new Array(6)
      while (index < 42) {
        result[resIndex++] = arr.slice(index, index += 7)
      }
      this.dates = result
    },
    getClasses (cell) {
      const classes = []
      const cellTime = cell.date.getTime()
      const curTime = this.value ? new Date(this.value).setHours(0, 0, 0, 0) : 0
      const startTime = this.startAt ? new Date(this.startAt).setHours(0, 0, 0, 0) : 0
      const endTime = this.endAt ? new Date(this.endAt).setHours(0, 0, 0, 0) : 0
      const today = new Date().setHours(0, 0, 0, 0)
      classes.push(cell.classes)
      if (cellTime === today) {
        classes.push('today')
      }
      if (cellTime > today && endTime) {
        classes.push('disabled')
      }
      if (cellTime === curTime) {
        classes.push('current')
      } else if (startTime) {
        if (cellTime < startTime) {
          classes.push('disabled')
        } else if (curTime && cellTime <= curTime && curTime >= startTime) {
          classes.push('inrange')
        }
      } else if (endTime) {
        if (cellTime > endTime) {
          classes.push('disabled')
        } else if (curTime && cellTime >= curTime) {
          classes.push('inrange')
        }
      }
      return classes.join(' ')
    },
    showMonths () {
      if (this.currentPanel === 'months') {
        this.currentPanel = 'date'
      } else {
        this.currentPanel = 'months'
      }
    },
    showYears () {
      // 当前年代
      if (this.currentPanel === 'years') {
        this.currentPanel = 'date'
      } else {
        let firstYear = Math.floor(this.now.getFullYear() / 10) * 10
        let years = []
        for (let i = 0; i < 10; i++) {
          years.push(firstYear + i)
        }
        this.years = years
        this.currentPanel = 'years'
      }
    },
    // 前进或后退一年
    changeYear (flag) {
      if (this.currentPanel === 'years') {
        this.years = this.years.map(v => v + flag * 10)
      } else {
        const now = new Date(this.now)
        now.setFullYear(now.getFullYear() + flag)
        this.now = now
      }
    },
    changeMonth (flag) {
      const now = new Date(this.now)
      now.setMonth(now.getMonth() + flag)
      this.now = now
    },
    selectDate (cell) {
      const classes = this.getClasses(cell)
      if (classes.indexOf('disabled') !== -1) {
        return
      }
      this.$emit('input', cell.date)
      this.$emit('select')
    },
    selectYear (year) {
      const now = new Date(this.now)
      now.setFullYear(year)
      this.now = now
      this.currentPanel = 'months'
    },
    selectMonth (month) {
      const now = new Date(this.now)
      now.setMonth(month)
      this.now = now
      this.currentPanel = 'date'
    }
  }
}

</script>

<style lang="stylus">

.calendar
  float left

.calendar *
  box-sizing border-box

.calendar a
  color inherit
  text-decoration none
  cursor pointer

.calendar-header
  line-height 34px
  text-align center

.calendar__next-icon
.calendar__prev-icon
  font-size 20px
  padding 0 6px

.calendar-header
  a
    &:hover
      color #1284e7

.calendar__prev-icon
  float left
.calendar__next-icon
  float right

.calendar-table
  width 100%
  font-size 12px
  table-layout fixed
  border-collapse collapse
  border-spacing 0

.calendar-table td
.calendar-table th
  width 32px
  height 32px
  text-align center

.calendar-table
  td
    cursor pointer

.calendar-table td.inrange
.calendar-table td:hover
.calendar-year  a:hover
.calendar-month  a:hover
  background-color #eaf8fe

.calendar-table td.current
.calendar-year  a.current
.calendar-month  a.current
  color #fff
  background-color #ec5222

.calendar-table td.disabled
  cursor not-allowed
  color #ccc
  background-color #f3f3f3

.lastMonth
.nextMonth
  color #ddd

.today
  color #da3838
.calendar-year
.calendar-month
  width 100%
  height 224px
  padding 7px 0
  text-align center

.calendar-year > a
  display inline-block
  width 40%
  margin 1px 5%
  line-height 40px

.calendar-month > a
  display inline-block
  width 30%
  line-height 40px
  margin 8px 1.5%

</style>
