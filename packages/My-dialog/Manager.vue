<template>
  <div
    ref="my-dialog__manager"
    :style="{ width: width + 'px', height: height + 'px' }"
    class="my-dialog__manager"
    :class="positionKeep ? 'my-dialog__manager__positionKeep' : 'my-dialog__manager__normal'"
  >
    <div @mousedown.self="beginMoveDialog" class="my-dialog__manager__header">
      <button
        :class="positionKeep ? 'positionKeep__open' : 'positionKeep__close'"
        @click="positionKeep = !positionKeep"
      >
        <i class="my-ui-x-iconfont icon-guding"></i>
      </button>
      <button @click="closeManager">
        <i class="my-ui-x-iconfont icon-guanbi"></i>
      </button>
    </div>
    <ul class="my-dialog__manager__body">
      <li
        class="my-dialog__manager__body__item"
        v-for="(item) in managerData"
        @click="showDialog(item.__vId)"
        :key="item.__vId"
      >
        {{ item.title }}
        <span @click.stop="closeDialog(item.__vId)" class="my-ui-x-iconfont icon-guanbi"></span>
      </li>
    </ul>
  </div>
</template>

<script>
const managerData = []
let toBeDeleted = {}
let addOneFlag = false
export default {
  props: {
    detection: {
      type: Boolean,
      default: true
    },
    width: {
      type: Number,
      default: 50
    },
    height: {
      type: Number,
      default: 50
    }
  },
  data() {
    return {
      managerData,
      positionKeep: false,
      timer: null
    };
  },
  watch: {
    managerData() {
      if (addOneFlag == true && !this.positionKeep) {
        this.addTransition()
      }
    }
  },
  mounted() {
    // 插入到body
    this.appendToWrap()
    //初始化位置
    this.initializePosition()
    //浏览器 缩放跟随
    this.screenSizeWatch()
  },
  methods: {
    //移动的方法
    beginMoveDialog(e) {

      this.$refs["my-dialog__manager"].style.transitionProperty = ' width, height '
      // 鼠标按下移动时禁止选中文字 
      let { offsetX, offsetY } = e;
      //是否需要移动功能
      document.documentElement.onmousemove = (e) => {
        let { clientX, clientY } = e;
        this.$refs["my-dialog__manager"].style.position = "fixed";
        // this.$refs['fl__dialog__wrap__container'].style.boxShadow = '0 0 10px 5px rgba(0,0,0,0.5)'
        this.$refs["my-dialog__manager"].style.left = clientX - offsetX + "px";
        this.$refs["my-dialog__manager"].style.top = clientY - offsetY + "px";
        if (this.detection) {
          this.collisionDetection(this.$refs["my-dialog__manager"])
        }
      };
      //添加鼠标松开监听
      document.documentElement.onmouseup = () => {
        document.documentElement.onmousemove = null;
        this.screenSizeWatch()
        this.$refs["my-dialog__manager"].style.transitionProperty = ' width, height, top, left'
      };
    },
    //碰撞检测
    collisionDetection(dialogManager) {
      if (parseInt(dialogManager.style.left) <= 0) {
        dialogManager.style.left = "0px";
      }
      if (parseInt(dialogManager.style.top) <= 0) {
        dialogManager.style.top = "0px";
      }
      if (parseInt(dialogManager.style.left) >= (window.innerWidth - 120)) {
        dialogManager.style.left = window.innerWidth - 120 + "px";
      }
      if (parseInt(dialogManager.style.top) >= window.innerHeight - 200) {
        dialogManager.style.top = window.innerHeight - 200 + "px";
      }
    },
    //浏览器 窗口缩放时自动更改位置
    screenSizeWatch() {
      //监听窗口缩放时自动更改位置
      let SL = window.innerWidth, ST = window.innerHeight
      setInterval(() => {
        let innerSL = window.innerWidth, innerST = window.innerHeight
        if (SL !== innerSL || ST !== innerST) {
          if (!this.isFullScreen) {
            this.initializePosition()
          }
        }
        SL = innerSL, ST = innerST
      }, 800);
    },
    //初始化位置
    initializePosition() {
      let dialogManager = document.querySelector(".my-dialog__manager");
      dialogManager.style.position = "fixed";
      dialogManager.style.top = "50px";
      dialogManager.style.left = window.innerWidth - parseInt(dialogManager.style.width) - 120 + "px";
    },
    // 插入到body
    appendToWrap() {
      return document.querySelector(".my-dialog__manager__wrap").append(document.querySelector(".my-dialog__manager"))
    },
    //点击li显示对应弹框
    showDialog(__vId) {
      managerData.forEach((item, index) => {
        if (item.__vId == __vId) {
          toBeDeleted.__vId = item.__vId
          managerData.splice(index, 1)
        }
      })

    },
    //关闭对应弹框
    closeDialog(__vId) {
      managerData.forEach((item, index) => {
        if (item.__vId == __vId) {
          managerData.splice(index, 1)
        }
      })
    },
    //处理固定事件
    closeManager() {
      managerData.splice(0, managerData.length)
    },
    addTransition() {
      clearTimeout(this.timer)
      this.positionKeep = true
      this.timer = setTimeout(() => {
        this.positionKeep = false
        addOneFlag = false
      }, 3000);
    }
  },
  managerData,
  toBeDeleted,
  addOne() {
    addOneFlag = true
  }

}
</script>

 
<style lang='less' scoped>
.my-dialog__manager__normal {
  &:hover {
    width: 120px !important;
    height: 200px !important;
  }
}
.my-dialog__manager__positionKeep {
  width: 120px !important;
  height: 200px !important;
}
.my-dialog__manager {
  font-size: 12px;
  line-height: 25px;
  position: fixed;
  z-index: 99999;
  display: none;
  flex-direction: column;
  margin: 0 auto;
  overflow: hidden;
  //防碰撞动画
  transition-property: width, height, top, left;
  transition-duration: 0.5s, 0.5s, 0.5s, 0.5s;
  border-radius: 3px;
  box-shadow: 0 4px 10px 0 rgba(0, 0, 0, 0.2);
  -moz-user-select: none;
  -khtml-user-select: none;
  user-select: none;
  min-width: 60px;
  &:hover &__header > :first-child {
    display: flex;
  }
  &:hover &__body {
    overflow: auto;
  }
  &__header {
    height: 50px;
    max-height: 30px;
    min-height: 20px;
    cursor: move;
    background-color: #dedede;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    .positionKeep__close {
      display: none;
    }
    .positionKeep__open {
      display: flex;
      background-color: #b2c4fc;
      border-color: #b2c4fc;
    }
    button {
      background-color: transparent;
      margin: 0 5px;
      color: #444;
      border: 8px solid transparent;
      width: 17px;
      height: 17px;
      display: flex;
      justify-content: center;
      align-items: center;
      border-radius: 50%;
      &:hover {
        background-color: #b2c4fc;
        border-color: #b2c4fc;
      }
      i {
        position: relative;
        top:  1px;
        font-size: 12px;
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
        height: 100%;
        transform: scale(0.8);
      }
    }
  }
  &__body {
    background-color: #fff;
    height: 170px;
    padding: 0 5px;
    overflow: hidden;
    &::-webkit-scrollbar {
      width: 4px;
      height: 4px;
    }
    &::-webkit-scrollbar-track {
      box-shadow: inset 0 0 0px rgba(235, 238, 241, 0.6);
    }
    &::-webkit-scrollbar-thumb {
      border-radius: 10px;
      box-shadow: inset 0 0 0px rgba(235, 238, 241, 0.6);
      background-color: rgba(180, 180, 180, 0.4);
    }

    &__item {
      white-space: nowrap;
      width: 100%;
      max-height: 25px;
      min-height: 25px;
      background-color: #eee;
      cursor: pointer;
      border: 1px solid #ccc;
      border-radius: 0.2em;
      margin: 3px 0;
      padding: 0 5px;
      overflow: hidden;
      text-overflow: ellipsis;
      position: relative;
      > span {
        display: none;
        font-size: 12px;
        color: #444;
        position: absolute;
        right: 0;
        top: 50%;
        transform: translateY(-50%) scale(0.8);
        width: 20px;
        height: 20px;
        text-align: center;
        line-height: 20px;
        border-radius: 50%;
        &:hover {
          opacity: 0.8;
        }
      }
      &:hover {
        opacity: 0.9;
        span {
          display: inherit;
        }
      }
    }
  }
}
</style>