<template>
    <div class="show-schedule" id="show-schedule" v-if='show'>
      <!-- 日历-bar -->
      <div class="content-block-title">
        <a href="javascript:void(0)" @click="lastMonth()">＜</a>
        {{showDate}}
        <a href="javascript:void(0)" @click="nextMonth()">＞</a>
      </div>
      <!-- 日历显示部分 -->
      <div>
        <div class="weeks">
          <table class="table-bordered">
            <thead>
              <tr>
                <td>日</td>
                <td>一</td>
                <td>二</td>
                <td>三</td>
                <td>四</td>
                <td>五</td>
                <td>六</td>
              </tr>
            </thead>
            <tbody id="dateTbody">
              <tr v-for="(sd, faIndexs) in showDates" :key="faIndexs">
                <td class="day-td"
                :class="{'no-month-day': dt.falg == 2}"
                v-for="(dt, chIndexs) in sd.data"
                :key="dt.show"
                @click="chooseDate(faIndexs, chIndexs, dt.date, dt.falg)">
                  <span class="tdDate" :id="`${faIndexs}-${chIndexs}`" :class="{'active': dt.activeFlag}">{{dt.show}}</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
</template>

<script>
export default {
  props: {
    show: {
      type: Boolean,
      default: true
    }
  },
  created () {
    this.getNowDate()
    this.activeFlagNumber = this.nowDays
    this.formatDay()
    this.handleDate()
    this.lastDate = this.nowDate
  },
  data () {
    return {
      showDate: '', // 视图上面用于显示的年和月
      showDates: [], // 视图上用于显示的日历数据
      showValue: false,
      finishDate: [], // formatDay之后的日期
      now: '', // 此时此刻的真实时间
      nowDays: '', // 今天的真实日
      nowDate: '', // 今天的真实日期
      chIndex: -1, // 用来以id处理active状态
      faIndex: -1, // 用来以id处理active状态
      nowYearAndMonth: '', // 今天真实的年月
      activeFlagNumber: 0, // 进入每个月的时候设定的值 可为当前日 或者 1
      lastDate: '' // 当前选中的日期
    }
  },
  methods: {
    /*eslint-disable */
    formatDay (timestr) {
      let t = /(\d+)-(\d+)-(\d+)\s*(\d*):?(\d*):?(\d*)/.exec(timestr)
      let da = ''
      let dn = this.getdate (this.now)
      if (t) {
        da = new Date(t[1], t[2]-1, 1, t[4], t[5], t[6])
      }
      else {
        da = new Date(dn['y'], dn['m'], 1, dn['h'], dn['i'], dn['s'])
      }
      this.date = this.getdate(da)
      let mon = [31, this.date['y'] % 4 == 0 ? 29 : 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
      let str = ''
      let day = 1
      let day2 = 0
      let dates = [this.date['y'], this.fillzero(this.date['m']+1)] // 获取当前年月
      this.finishDate = [] // 初始化
      this.showDate = `${dates[0]}-${dates[1]}`
      for(let md = mon[this.date['m']], wd = md-this.date['w'] + 1, n = 0; n < 6; n++){
        for(let nn = 0; nn < 7; nn++){
          if(wd <= md){ // 上个月
            let obj = {}
            /**
             * [date description]
             * @type {String}
             * 解释一下obj吧。。维护的时候比较容易
             * date: 对应的当前日
             * flag: 是不是本月(表格显示用的-不是本月灰色，是本月白色)
             * show: 为了方便, 用于显示的具体日(2016-01-01 就是显示 01)
             */
            obj.date = ''
            obj.falg = 2
            obj.show = wd
            this.finishDate.push(obj)
            wd++
          }
          else {
            if(day <= mon[this.date['m']]){
              let obj = {}
              let goDay = day++
              let fixGoDay = goDay
              if (fixGoDay < 10) {
                fixGoDay = '0' + fixGoDay
              }
              let activeFlag = false
              if (this.activeFlagNumber == fixGoDay) {
                // 把默认的选中的当前月的当前天或者某个月的第一天 传给this，以便于点击其他日期的时候给默认的activeFlag置为false
                this.faIndex = n
                this.chIndex = nn
                activeFlag = true
                this.lastDate = `${this.showDate}-${fixGoDay}`
              }
              else {
                activeFlag = false
              }
              obj.date = `${dates[0]}-${dates[1]}-${fixGoDay}`
              obj.falg = 1
              obj.show = goDay
              obj.activeFlag = activeFlag
              this.finishDate.push(obj)
              day2 = 1
            }
            else{
              // 下一个月的
              let obj = {}
              obj.date = ''
              obj.falg = 2
              obj.show = day2
              this.finishDate.push(obj)
              day2++
            }
          }
        }
      }
    },
    // 获取当前日期
    getNowDate () {
      this.now = new Date()
      let nowMonth = this.now.getMonth() + 1
      this.nowDays = this.now.getDate() // 获取当天的日 后面计算用到(遍历日历模板时，默认选中active状态)
      // 获取当前年月，用于获取当前月的有日程的日期
      this.nowYearAndMonth = this.now.getFullYear() + '-' + ((nowMonth > 9) ? nowMonth : ('0' + nowMonth))
      // 当天的日期，首次进入应用默认获取当签日期的日期
      this.nowDate = this.now.getFullYear() + '-' + ((nowMonth > 9) ? nowMonth : ('0' + nowMonth)) + '-' + ((this.nowDays > 9) ? this.nowDays : ('0' + this.nowDays))
    },
    handleDate () {
      let temp = [
        {
          data: this.finishDate.slice(0, 7)
        },
        {
          data: this.finishDate.slice(7, 14)
        },
        {
          data: this.finishDate.slice(14, 21)
        },
        {
          data: this.finishDate.slice(21, 28)
        },
        {
          data: this.finishDate.slice(28, 35)
        },
        {
          data: this.finishDate.slice(35, 42)
        }
      ]
      // 某一行不是本月的去掉
      let tr = [temp[0], temp[5]]
      let first = false
      let last = false
      tr.forEach((e, index) => {
        e.data.forEach(ele => {
          if (ele.falg == 1) {
            index == 0 ? first = true : last = true
          }
        })
      })
      if (!first) {
        temp.shift()
      }
      if (!last) {
        temp.pop()
      }
      this.showDates = temp
    },
    getDateYM (data) {
      let dn = this.getdate(data)
      let da = new Date(dn['y'], dn['m'], 1, dn['h'], dn['i'], dn['s'])
      this.date = this.getdate(da)
      let dates = [this.date['y'], this.fillzero(this.date['m'] + 1)] // 获取当前年月
      return `${dates[0]}-${dates[1]}`
    },
    // 获取时间
    getdate (da) {
      let date = []
      date['y'] = da.getFullYear()
      date['m'] = da.getMonth()
      date['d'] = da.getDate()
      date['w'] = da.getDay()
      date['h'] = da.getHours()
      date['i'] = da.getMinutes()
      date['s'] = da.getSeconds()
      return date
    },
    // 上个月
    lastMonth () {
      this.now.setMonth(this.date['m'] - 1)
      // 判断是不是这个月，是了要默认到当前天啊。。。。
      if (this.getDateYM(this.now) === this.nowYearAndMonth) {
        this.activeFlagNumber = +this.nowDays
      }
      else {
        this.activeFlagNumber = 1
      }
      // todo faIndex 重设
      this.formatDay()
      this.handleDate()
      // this.lastDate = this.nowDate
    },
    // 下个月
    nextMonth () {
      this.now.setMonth(this.date['m'] + 1)
      // 判断是不是这个月，是了要默认到当前天啊。。。。
      if (this.getDateYM(this.now) === this.nowYearAndMonth) {
        this.activeFlagNumber = +this.nowDays
      }
      else {
        this.activeFlagNumber = 1
      }
      this.formatDay()
      this.handleDate()
      // this.lastDate = this.nowDate
    },
    //如果日期为一位数，则在前面补零
    fillzero (str) {
      let strs
      strs = typeof str == 'string' ? str : str.toString() //eslint-disabled-line
      if(strs.length == 1){
        strs = '0' + strs
      }
      return strs
    },
    // 点击选择日期
    chooseDate (faIndexs, chIndexs, data, flag) {
      // flag为2的时候代表不是当前月的日子 点击无效
      if (flag === 2) {
        return
      }
      // 先把之前的选中的置为false 再把选中的置为true 再把选中的传给this
      this.showDates[this.faIndex].data[this.chIndex].activeFlag = false
      this.showDates[faIndexs].data[chIndexs].activeFlag = true
      this.faIndex = faIndexs
      this.chIndex = chIndexs
      this.lastDate = data
    }
  },
  watch: {
    lastDate (val) {
      this.$emit('on-change', val)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
a {
  text-decoration: none;
}
.table-bordered {
  border: 0;
  width: 100%;
  background-color: transparent;
  color:#666666;
  margin: 0 auto;
}
.no-month-day {
  color: #999;
}
.tdDate {
  width: 1.7rem;
  border-radius: 1.7rem;
  line-height: 1.7rem;
  height: 1.7rem;
  display: inline-block;
}
.active {
  background-color: #fc6980;
  color: #fff !important;
}
</style>
