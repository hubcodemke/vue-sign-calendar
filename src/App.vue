<template>
  <div id="app">
    <signCalendar
      :open="calendar.open"
      :unclick="calendar.unClick"
      :showtop="calendar.showTop"
      :signeddates="calendar.signedDates"
    ></signCalendar>
    <div class="txt-c">
      <button @click.stop="repairSignDialog=true" class="btn">立即补签</button>
    </div>
    <!-- 补签 -->
    <my-dialog
      :visible.sync="repairSignDialog"
      :showheader="false"
      :showfooter="false"
      :closeonclickmodal="false"
      :closeimg="require('@/assets/img/close-1.png')"
      top="10%"
    >
      <div class="repair-sign-dialog">
        <signCalendar
          :showopenbtn="false"
          :signeddates="calendar.signedDates"
          :maxchoose="calendar.maxChoose"
          :choosecallback="chooseHandle"
        ></signCalendar>
        <div class="fbar">
          <span>{{chooseDatesText}}</span>
          <span>可补签{{calendar.maxChoose}}次</span>
        </div>
        <div class="txt-c">
          <button class="btn" @click="repairSignHandle">确认补签</button>
        </div>
      </div>
    </my-dialog>
  </div>
</template>

<script>
import myDialog from "./components/myDialog.vue";
import signCalendar from "./components/signCalendar.vue";

export default {
  name: "app",
  data() {
    return {
      calendar: {
        open: false,
        unClick: true,
        showTop: false,
        maxChoose: 5, //补签次数
        signedDates: ["2019-10-1", "2019-10-2", "2019-10-6", "2019-10-11"]
      },
      repairSignDialog: false,
      chooseDatesText: "选择补签日期",
      chooseDates: []
    };
  },
  methods: {
    //选择补签日期回调
    chooseHandle(data) {
      if (data.length > 0) {
        this.chooseDates = data;
        let str = "";
        data.forEach(v => {
          str = str + this.dateFormat(v, "d") + "、";
        });
        this.chooseDatesText = `选中${str.substring(0, str.length - 1)}号补签`;
      } else {
        this.chooseDatesText = "选择补签日期";
      }
    },
    //确认补签
    repairSignHandle() {
      if (this.chooseDates.length > 0) {
        //TODO
      } else {
        alert("请先选择补签日期");
      }
    },
    dateFormat(d, fmt) {
      let D = new Date(d);
      let o = {
        "M+": D.getMonth() + 1, //月份
        "d+": D.getDate(), //日
        "h+": D.getHours(), //小时
        "m+": D.getMinutes(), //分
        "s+": D.getSeconds(), //秒
        "q+": Math.floor((D.getMonth() + 3) / 3), //季度
        S: D.getMilliseconds() //毫秒
      };
      if (/(y+)/.test(fmt))
        fmt = fmt.replace(
          RegExp.$1,
          (D.getFullYear() + "").substr(4 - RegExp.$1.length)
        );
      for (let k in o)
        if (new RegExp("(" + k + ")").test(fmt))
          fmt = fmt.replace(
            RegExp.$1,
            RegExp.$1.length == 1
              ? o[k]
              : ("00" + o[k]).substr(("" + o[k]).length)
          );
      return fmt;
    }
  },
  components: {
    signCalendar,
    myDialog
  }
};
</script>

<style lang="scss">
* {
  margin: 0;
  padding: 0;
}
.txt-c {
  text-align: center;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.btn {
  margin: 0 auto;
  padding: 0.1rem;
}
.repair-sign-dialog {
  border-radius: 5px;
  overflow: hidden;
  .fbar {
    border-top: 1px solid rgba(255, 255, 255, 0.3);
    background-color: #1c7ce5;
    justify-content: space-between;
    font-size: 0.26rem;
    color: #c6e1ff;
    height: 0.8rem;
    line-height: 0.8rem;
    padding: 0 0.34rem;
    display: flex;
    align-items: center;
  }
  .btn {
    margin: 5px;
  }
  .signCalendar {
    padding-bottom: 0 !important;
    .week {
      margin-bottom: 0 !important;
    }
  }
}
</style>
