<template>
  <div>
    <el-switch v-model="multiType" active-text="取消勾选" inactive-text="不做处理"/>
    <div class="mainDiv" ref="mainDiv" @mousedown.right.prevent="slideBegin" @contextmenu.prevent="()=>{}">
      <el-table :data="data" ref="table" @selection-change="selectionChange" :select-on-indeterminate="false">
        <el-table-column type="selection" width="55"/>
        <el-table-column prop="seatNo" min-width='74px' label="自编号" />
        <el-table-column prop="code" label="标本编号" min-width='118px' />
        <el-table-column prop="name" label="姓名" min-width="65px" />
        <el-table-column prop="inspectProjectType" label="套餐名称" min-width='95px' type="BasComboInfoMaster"/>
        <el-table-column prop="age" label="年龄" min-width='53px' />
      </el-table>
      <div class="slider" :style="`top:${sTop}px; left:${sLeft}px; height:${sHeight}px; width:${sWidth}px;`" v-show="sliding"/>
    </div>
  </div>
</template>

<script>
import dataList from './data.js'
export default {
  name: 'testTable',
  props: {
    data: {
      type: Array,
      default:() => dataList
    }
  },
  data() {
    return {
      multiType: false,
      sliding: false,
      sliderData: {
        parentX: 0,
        parentY: 0,
        beginX: 0,
        beginY: 0,
        endX: 0,
        endY: 0
      },
      selectionList: [],
    }
  },
  computed: {
    // 位置与宽高
    sTop() {
      return (this.sliderData.beginY < this.sliderData.endY ? this.sliderData.beginY : this.sliderData.endY) - this.sliderData.parentY;
    },
    sLeft() {
      return (this.sliderData.beginX < this.sliderData.endX ? this.sliderData.beginX : this.sliderData.endX) - this.sliderData.parentX;
    },
    sHeight() {
      return Math.abs(this.sliderData.beginY - this.sliderData.endY);
    },
    sWidth() {
      return Math.abs(this.sliderData.beginX - this.sliderData.endX);
    },
    // 底部位置
    sBottom() {
      return this.sTop + this.sHeight;
    }
  },
  methods: {
    // 鼠标右键按下
    slideBegin(e) {
      this.sliderData.parentX = this.$refs.mainDiv.offsetLeft;
      this.sliderData.parentY = this.$refs.mainDiv.offsetTop;
      this.sliderData.beginX = e.pageX;
      this.sliderData.beginY = e.pageY;
      this.sliderData.endX = e.pageX;
      this.sliderData.endY = e.pageY;
      document.addEventListener('mousemove', this.slideCourse, { passive: false });
      document.addEventListener('mouseup', this.slideEnd, { passive: false });
    },
    // 鼠标拖动
    slideCourse(e) {
      e.preventDefault();
      e.stopPropagation();
      this.sliderData.endX = e.pageX;
      this.sliderData.endY = e.pageY;
      if(this.sHeight > 5 && this.sWidth > 5) {
        this.sliding = true;
      }
    },
    // 鼠标右键放开
    slideEnd() {
      document.removeEventListener('mousemove', this.slideCourse, { passive: false });
      document.removeEventListener('mouseup', this.slideEnd, { passive: false });
      if(!this.sliding) return;
      this.sliding = false;
      let theadHeight = this.$refs.table.$el.getElementsByTagName('thead')[0].offsetHeight;
      let divs = [...this.$refs.table.$el.getElementsByTagName('tbody')[0].children];
      // 是否继续判断逻辑
      let inSlider = true;
      // 被框选元素起止地址
      let beginInd, endInd;
      divs.forEach((ele, ind) => {
        // 起止地址同时不为空时，不再进行判断
        if(!inSlider) {
          return;
        }
        // 未进入范围，查找头
        if(beginInd === undefined && endInd === undefined) {
          console.log(ele.offsetTop, ele.offsetHeight, this.sTop, this.sBottom);
          if(ele.offsetTop + ele.offsetHeight + theadHeight >= this.sTop) {
            beginInd = endInd = ind;
          }
        }
        // 已进入范围，查找尾
        else {
          endInd = ind;
          if(ele.offsetTop + theadHeight > this.sBottom) {
            inSlider = !inSlider;
          }
        }
      });
      this.selectByInd(beginInd, endInd);
    },
    // 更改选中状态
    selectByInd(beginInd, endInd) {
      this.data.slice(beginInd, endInd)
      .forEach((item) => {
        if(this.multiType) {
          this.$refs.table.toggleRowSelection(item);
        }
        else {
          this.$refs.table.toggleRowSelection(item, true);
        }
      })
    },
    selectionChange(val) {
      this.selectionList = val;
    },
    getIsNull(val) {
      return val === undefined || val === null || val === 0;
    }
  }
}
</script>

<style lang="scss" scoped>
.mainDiv {
  user-select: none;
  position: relative;
  overflow: hidden;
}
.slider {
  border: 1px solid #409EFF;
  background-color: #6CB6FF38;
  position: absolute;
  z-index: 1000;
}
</style>
