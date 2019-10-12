<!-- 
   name:自定义签到日历组件
   date:2019-8-16
   des:用来操作当月签到信息，补签等
 -->
<template>
  <div class="signCalendar">
    <div class="top-bar" v-show="showtop">{{y}}{{text.year}}{{m+1}}{{text.month}}</div>
    <div class="week" :style="showtop ? 'border-bottom:0' :'' ">
      <span class="week-day" v-for="(item,index) in weekDay" :key="index">{{item}}</span>
    </div>
    <div
      :class="{'hide' : !monthOpen}"
      class="content"
      :style="{
        height: (dates.length / 7) * 40 + 'px'
      }"
    >
      <div :style="{ top : positionTop + 'px' }" class="days">
        <span
          v-for="(item,index) in dates"
          :key="index"
          :class="{
              nolm: !item.lm,
              signed:isSigned(item.year, item.month+1, item.date),
              today: isToday(item.year, item.month, item.date)
           }"
        >
          <em @click.stop="!unclick ? selectOne(item,$event):''">{{item.date}}</em>
          <i></i>
          <b v-if="isToday(item.year, item.month, item.date)">今</b>
        </span>
      </div>
    </div>
    <div @click="trgWeek()" :class="{down:!monthOpen}" class="weektoggel" v-if="showopenbtn"></div>
  </div>
</template>

<script>
export default {
  name: "signCalendar",
  props: {
    //第一列星期几
    weekstart: {
      type: Number,
      default: 7
    },
    //已经签到的日期
    signeddates: {
      type: Array,
      default: null
    },
    //最大可选择数
    maxchoose: {
      type: Number,
      default: 0
    },
    //选择回调
    choosecallback: {
      type: Function,
      default() {
        return () => {};
      }
    },
    //是否展开
    open: {
      type: Boolean,
      default: true
    },
    //显示顶部
    showtop: {
      type: Boolean,
      default: true
    },
    //显示底部伸缩按钮
    showopenbtn: {
      type: Boolean,
      default: true
    },
    //禁用点击事件
    unclick: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      el: null,
      text: {
        year: "年",
        month: "月",
        week: ["一", "二", "三", "四", "五", "六", "日"],
        today: "今"
      },
      y: new Date().getFullYear(), //年
      m: new Date().getMonth(), //月
      dates: [], //当前月日期集合
      positionTop: 0,
      monthOpen: true,
      chooseDates: [] //选中的日期集合
    };
  },
  created() {
    this.dates = this.monthDay(this.y, this.m);
    !this.open && this.trgWeek();
  },
  mounted() {
    this.el = document.querySelector(".signCalendar");
  },
  computed: {
    //顶部星期栏目
    weekDay() {
      return this.text.week
        .slice(this.weekstart - 1)
        .concat(this.text.week.slice(0, this.weekstart - 1));
    }
  },
  methods: {
    //小于10格式化
    // zero(n) {
    //   return n < 10 ? "0" + n : n;
    // },
    //获取当前月份天数
    monthDay(y, m) {
      let firstDayOfMonth = new Date(y, m, 1).getDay(); // 当月第一天星期几
      let lastDateOfMonth = new Date(y, m + 1, 0).getDate(); // 当月最后一天
      let lastDayOfLastMonth = new Date(y, m, 0).getDate(); // 上一月的最后一天
      let dates = []; // 所有渲染日历
      let weekstart = this.weekstart == 7 ? 0 : this.weekstart; // 方便进行日期计算，默认星期从0开始

      let startDay = (() => {
        // 周初有几天是上个月的
        if (firstDayOfMonth == weekstart) {
          return 0;
        } else if (firstDayOfMonth > weekstart) {
          return firstDayOfMonth - weekstart;
        } else {
          return 7 - weekstart + firstDayOfMonth;
        }
      })();
      let endDay = 7 - ((startDay + lastDateOfMonth) % 7); // 结束还有几天是下个月的
      for (let i = 1; i <= startDay; i++) {
        dates.push({
          date: lastDayOfLastMonth - startDay + i,
          day: weekstart + i - 1 || 7,
          month: m - 1 >= 0 ? m - 1 : 12,
          year: m - 1 >= 0 ? y : y - 1
        });
      }
      for (let j = 1; j <= lastDateOfMonth; j++) {
        dates.push({
          date: j,
          day: (j % 7) + firstDayOfMonth - 1 || 7,
          month: m,
          year: y,
          lm: true
        });
      }
      for (let k = 1; k <= endDay; k++) {
        dates.push({
          date: k,
          day: (lastDateOfMonth + startDay + weekstart + k - 1) % 7 || 7,
          month: m + 1 <= 11 ? m + 1 : 0,
          year: m + 1 <= 11 ? y : y + 1
        });
      }
      return dates;
    },
    //已经签到处理
    isSigned(y, m, d) {
      let flag = false;
      for (let i = 0; i < this.signeddates.length; i++) {
        let dy = `${y}-${m}-${d}`;
        if (this.signeddates[i] == dy) {
          flag = true;
          break;
        }
      }
      return flag;
    },
    isToday(y, m, d) {
      let date = new Date();
      return (
        y == date.getFullYear() && m == date.getMonth() && d == date.getDate()
      );
    },
    //切换成周模式
    trgWeek() {
      this.monthOpen = !this.monthOpen;
      if (this.monthOpen) {
        this.positionTop = 0;
      } else {
        let index = -1;
        this.dates.forEach((i, x) => {
          this.isToday(i.year, i.month, i.date) && (index = x);
        });
        this.positionTop = -((Math.ceil((index + 1) / 7) || 1) - 1) * 40;
      }
    },
    //点击回调
    selectOne(i, event) {
      let dy = `${i.year}-${i.month + 1}-${i.date}`;
      let firstD = new Date(this.y, this.m, 1);
      let todayD = new Date();
      let selectD = new Date(dy);
      if (
        selectD < firstD ||
        selectD.getDate() >= todayD.getDate() ||
        this.signeddates.includes(dy)
      ) {
        //console.log("不在可选范围内")
        return false;
      }
      //如果已存在就删除，不存在则添加
      let ind = this.chooseDates.findIndex(value => {
        return value == dy;
      });
      var parDom = event.srcElement.parentNode;
      if (
        ind === -1 &&
        (this.chooseDates.length < this.maxchoose || this.maxchoose === 0)
      ) {
        this.chooseDates.push(dy);
        parDom.classList.add("choose");
      } else if (ind > -1) {
        this.chooseDates.splice(ind, 1);
        parDom.classList.remove("choose");
      }
      //排序
      this.chooseDates.sort((a, b) => {
        return new Date(a).getTime() - new Date(b).getTime();
      });
      this.choosecallback(this.chooseDates);
    }
  }
};
</script>

<style lang="scss" scoped>
.signCalendar {
  color: #fff;
  font-size: 0.26rem;
  text-align: center;
  background-color: #1c7ce5;
  padding-bottom: 0.16rem;
  .top-bar {
    font-size: 0.32rem;
    height: 0.88rem;
    line-height: 0.88rem;
    border-bottom: 1px solid rgba(255, 255, 255, 0.3);
  }
  .week {
    display: flex;
    align-items: center;
    height: 0.96rem;
    line-height: 0.96rem;
    border-bottom: 1px solid rgba(255, 255, 255, 0.2);
    margin-bottom: 0.15rem;
    span {
      flex: 1;
    }
  }
  .content {
    position: relative;
    overflow: hidden;
    transition: height 0.4s ease;
    .days {
      transition: top 0.3s;
      display: flex;
      align-items: center;
      flex-wrap: wrap;
      position: relative;
      span {
        position: relative;
        display: block;
        height: 40px;
        line-height: 40px;
        width: calc(100% / 7);
        em {
          font-style: normal;
          display: inline-block;
          vertical-align: middle;
          width: 0.45rem;
          height: 0.45rem;
          line-height: 0.45rem;
          overflow: hidden;
          border-radius: 99em;
        }
        i {
          font-style: normal;
          display: none;
          width: 0.2rem;
          height: 0.2rem;
          background: url("../assets/img/ico-hook.png");
          background-size: 100% 100%;
          position: absolute;
          right: 0.22rem;
          bottom: 0.1rem;
        }
        b {
          font-size: 0.24rem;
          font-weight: normal;
          position: absolute;
          right: 0.05rem;
          top: -0.2rem;
        }
      }
      .choose {
        em {
          background-color: #9fcdff;
          color: #0157d8;
        }
      }
      .signed {
        em {
          background-color: rgba(0, 0, 0, 0.1);
        }
        i {
          display: block;
        }
      }
      .today {
        em {
          background-color: #fff;
          color: #0157d8;
        }
      }
      .nolm {
        em {
          color: #fff;
          opacity: 0.3;
        }
      }
    }
  }
  .hide {
    height: 40px !important;
  }
  .weektoggel {
    width: 0.8rem;
    height: 0.32rem;
    background: url("../assets/img/ico-arrow-up.png");
    background-size: 100% 100%;
    margin: 0.2rem auto 0;
  }
  & .down {
    background: url("../assets/img/ico-arrow-down.png");
    background-size: 100% 100%;
  }
}
</style>