<!-- 
   name:自定义对话框组件
   date:2019-8-16
   des:自定义输入内容等
 -->
<template>
  <transition name="fade">
    <div
      v-show="visible"
      :style="{backgroundColor: !this.modal ? 'transparent' : 'rgba(0, 0, 0, 0.6)'}"
      :class="customclass"
      @click.stop="modalClick"
      class="my-dialog"
    >
      <div
        :style="{width, top,
          paddingBottom:!showfooter ? '0':'1.1rem',
          borderRadius:radius ? '8px':'0',
          }"
        @click.stop
        class="dialogbox"
      >
        <em
          v-if="showclose"
          @click="close"
          :style="(closeimg != null?'background:url('+closeimg+');background-size:100% 100%;':'')"
          :class="{closeout}"
          class="close"
        ></em>
        <div
          v-if="showheader"
          :style="{textAlign:center ? 'center':'',
            borderRadius:radius ? '8px 8px 0 0':'0'}"
          class="header"
        >
          <slot name="header">{{title}}</slot>
        </div>
        <div class="main">
          <slot></slot>
        </div>
        <div
          v-if="showfooter"
          :style="{textAlign:center ? 'center':'',
            borderRadius:radius ? '0 0 8px 8px':'0'}"
          class="footer"
        >
          <slot name="footer"></slot>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
export default {
  name: "myDialog",
  props: {
    //头部标题
    title: {
      type: String,
      default: "提示"
    },
    //头部显示
    showheader: {
      type: Boolean,
      default: true
    },
    //底部显示
    showfooter: {
      type: Boolean,
      default: true
    },
    //是否显示
    visible: {
      type: Boolean,
      default: false
    },
    //宽度
    width: {
      type: String,
      default: "90%"
    },
    //是否需要遮罩层
    modal: {
      type: Boolean,
      default: true
    },
    //Dialog 的自定义类名
    customclass: {
      type: String,
      default: ""
    },
    //显示关闭按钮
    showclose: {
      type: Boolean,
      default: true
    },
    //通过点击 modal 关闭 Dialog
    closeonclickmodal: {
      type: Boolean,
      default: true
    },
    //关闭按钮在外部
    closeout: {
      type: Boolean,
      default: false
    },
    //关闭图片
    closeimg: {
      type: String,
      default: null
    },
    //关闭后回调函数
    closecallback: {
      type: Function
    },
    //关闭前的回调
    beforeclose: {
      type: Function
    },
    //头部和底部采用居中布局
    center: {
      type: Boolean,
      default: false
    },
    //边框圆角
    radius: {
      type: Boolean,
      default: true
    },
    //距离顶部位置
    top: {
      type: String,
      default: "15%"
    }
  },
  data() {
    return {};
  },
  methods: {
    //关闭
    close() {
      //关闭前回调
      if (this.beforeclose) {
        this.beforeclose(() => {
          this.$emit("update:visible", false);
          if (this.closecallback) {
            this.closecallback();
          }
        });
        return false;
      }

      if (this.closecallback) {
        this.closecallback();
      }
      this.$emit("update:visible", false);
    },
    modalClick() {
      this.closeonclickmodal ? this.close() : "";
    }
  }
};
</script>

<style lang="scss" scoped>
$footerHeight: 1.1rem;
.my-dialog {
  position: fixed;
  width: 100%;
  height: 100%;
  z-index: 9990;
  top: 0;
  left: 0;
  background-color: #000;
  background-color: rgba(0, 0, 0, 0.6);
  //   display: none;
  .dialogbox {
    position: relative;
    width: 6.8rem;
    min-height: 3.5rem;
    margin: 0 auto;
    top: 15%;
    color: #333;
    background-color: #fff;
    box-shadow: 0 10px 10px rgba(186, 131, 0, 0.1);
    padding-bottom: $footerHeight;
    .close {
      width: 0.3rem;
      height: 0.3rem;
      display: block;
      position: absolute;
      right: 0.3rem;
      top: 0.3rem;
      background: url("../assets/img/close.png");
      background-size: 100% 100%;
    }
    .closeout {
      width: 0.9rem;
      height: 0.9rem;
      background: url("../assets/img/closeout.png");
      background-size: 100% 100%;
      top: auto;
      bottom: -1.5rem;
      right: 50%;
      transform: translate(50%);
    }
  }
  .header {
    padding: 0.3rem;
    font-size: 0.35rem;
    line-height: 0.4rem;
  }
  .main {
    font-size: 0.26rem;
  }
  .footer {
    text-align: right;
    font-size: 0.28rem;
    background-color: #fff;
    padding: 0 2%;
    min-height: $footerHeight;
    line-height: $footerHeight;
    position: absolute;
    width: 96%;
    bottom: 0;
    left: 0;
  }
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>