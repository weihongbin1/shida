<template>
  <div class="components-attr-edit components-attr-animate-edit">
    <el-scrollbar class="components-attr-edit">
      <div class="attr-edit-inner" v-if="activeElementUUID">
        <div class="animate-edit-btn-wrapper">
          <el-button type="primary" icon="el-icon-plus" size="small" @click="addAnimate">添加动画</el-button>
          <el-button icon="el-icon-caret-right" size="small" @click="runAnimate(undefined)">预览动画</el-button>
        </div>
        <div class="el-animate-list-wrapper paddingT20" v-show="activeElement.animations.length">
          <el-collapse accordion>
            <el-collapse-item v-for="(item, index) in activeElement.animations" :key="index">
              <template slot="title">
                <span class="el-animate-title-name">动画 {{ index }}</span>
                <div class="el-animate-title-type-wrapper">
                  <span class="el-animate-title-type" @click.stop.prevent="handleShowChooseAnimatePanel(index)"
                    >{{ item.type }} <i class="el-icon-caret-right size-mini"></i>
                  </span>
                </div>
                <span class="el-animate-title-btn" @click.stop.prevent="runAnimate(index)"
                  ><i class="el-icon-caret-right"></i
                ></span>
                <span class="el-animate-title-btn" @click.stop.prevent="handleDeleteAnimate"
                  ><i class="el-icon-delete"></i
                ></span>
              </template>

              <div class="el-animate-item">
                <div class="attr-item-edit-wrapper">
                  <p class="attr-item-title">动画时长：</p>
                  <div class="col-2 attr-item-edit-input">
                    <el-input-number
                      size="mini"
                      v-model="item.duration"
                      controls-position="right"
                      :min="0"
                      :step="0.1"
                    />
                  </div>
                </div>
                <div class="attr-item-edit-wrapper">
                  <p class="attr-item-title">动画延迟：</p>
                  <div class="col-2  attr-item-edit-input">
                    <el-input-number size="mini" v-model="item.delay" controls-position="right" :min="0" :step="0.1" />
                  </div>
                </div>
                
              </div>
            </el-collapse-item>
          </el-collapse>
        </div>
      </div>
      <div v-else>
        <p class="gray paddingT30 text-center">请在画板上选择需要编辑得元素</p>
      </div>
    </el-scrollbar>

    <div
      class="components-attr-edit animate-choose-list-wrapper"
      :class="{ fadeInUp: showAnimatePanel, fadeInDown: !showAnimatePanel, animate: showAnimatePanel }"
    >
      <el-tabs v-model="activeName">
        <el-tab-pane v-for="item in animateCssDatas" :key="item.label" :label="item.label" :name="item.label">
          <el-scrollbar class="animate-choose-item">
            <div
              :class="animate.unclick?'animate-choose-item-inner unclick':'animate-choose-item-inner'"
              v-for="(animate, index) in item.children"
              :key="index"
              @mouseover="hoverPreviewAnimate = animate.value"
              @mouseleave="hoverPreviewAnimate = ''"
              @click="handleChooseAnimate(animate)"
            >
              <span
                class="animate-choose-image"
                :style="{ backgroundPosition: `${item.bg_X}px ${item.bg_Y}px` }"
                :class="[hoverPreviewAnimate === animate.value  && animate.value + ' animated']"
              ></span>
              <p>{{ animate.label }}</p>
            </div>
          </el-scrollbar>
        </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<script>
import animateCssData from "@client/common/animateCssData";
import filterCssData from "@client/common/ffcreatorCssData";
import { mapState, mapGetters } from "vuex";
import $bus from "@client/eventBus";
import {random} from 'lodash'

export default {
  data() {
    return {
      animateCssDatas: animateCssData,
      activeName: "进入",
      showAnimatePanel: false,
      reSelectAnimateIndex: undefined,
      hoverPreviewAnimate: "",
    };
  },
  computed: {
    ...mapState({
      projectData: state => state.editor.projectData,
      activePageUUID: state => state.editor.activePageUUID,
      activeElementUUID: state => state.editor.activeElementUUID
    }),
    ...mapGetters(["currentPageIndex", "activeElementIndex", "activeElement"]),
  },
  watch: {
    activePageUUID() {
      // 关闭选择动画弹窗
      this.addAnimate(false);
    },
    activeElementUUID() {
      // 关闭选择动画弹窗
      this.addAnimate(false);
    }
  },

  methods: {
    /**
     * 选取animate
     * @param item
     */
    handleChooseAnimate(item) {
	for(let i = 0;i < filterCssData.children.length;i++) {
        if(item.value === filterCssData.children[i].value) {
          console.log(item.value)
          return ''
        } 
      }
      this.showAnimatePanel = false;
      let randomAnimate
      if(item.value === "random") {
	console.log(item.value);
         randomAnimate = this.createRandomAnimate(this.activeName)
        item = randomAnimate
      }
      if (this.reSelectAnimateIndex === undefined) {
	console.log(item.value);
        this.$store.dispatch("addElementAnimate", item.value);
      } else {
        this.activeElement.animations[this.reSelectAnimateIndex].type = item.value;
        this.$store.dispatch("addHistoryCache");
      }
    },
    /**
     * 删除动画
     * @param index
     */
    handleDeleteAnimate(index) {
      this.$store.dispatch("deleteElementAnimate", index);
    },
    addAnimate(showAnimatePanel = true) {
      this.showAnimatePanel = showAnimatePanel;
      this.reSelectAnimateIndex = undefined;
    },
    handleShowChooseAnimatePanel(index) {
      this.reSelectAnimateIndex = index;
      this.showAnimatePanel = true;
    },
    /**
     * 执行此条动画效果
     */
    runAnimate(index) {
      console.log(index)
      let animationData = index === undefined ? this.activeElement.animations : [this.activeElement.animations[index]];
      $bus.$emit("RUN_ANIMATIONS", this.activeElement.uuid, animationData);
    },
    /**
     * 随机动画
     */
    createRandomAnimate(activeName){
      let animate
      if(activeName === "进入"){
         animate = animateCssData[0].children
      }else if(activeName === "强调"){
         animate = animateCssData[1].children
      }else if(activeName === "退出"){
         animate = animateCssData[2].children
      } else {
        return -1
      }
      const animateCounts = animate.length
	console.log(animateCounts)
      return animate[random(1, animateCounts - 1)]
    },
    //判断动画是否可以选择
    isdisClick(item) {
      for(let i = 0;i < filterCssData.children.length;i++) {
        if(item.value === filterCssData.children[i].value) {
          return ''
        } 
      }
      this.handleChooseAnimate(item)
    }
  }
};
</script>

<style lang="scss" scoped>
.components-attr-edit {
  height: 100%;
}

.components-attr-animate-edit {
  position: relative;
}

.attr-title {
  font-weight: bold;
}

.attr-item-edit-wrapper {
  padding-left: 4px;
  display: flex;
  width: 100%;
  text-align: center;
  padding-bottom: 10px;
  .attr-item-title {
    text-align: left;
    min-width: 60px;
    font-size: 12px;
    line-height: 28px;
  }
  .attr-item-edit-input {
    &.col-2 {
      width: 90px;
      margin-left: 10px;
    }
    &.col-1 {
      width: 250px;
    }
    &.col-3 {
      width: 60px;
      margin-left: 10px;
    }
    &.col-4 {
      width: 50px;
      margin-left: 10px;
    }
    .attr-item-edit-input-des {
      text-align: center;
      line-height: 1;
      margin-top: 2px;
      font-size: 12px;
      color: $gray;
    }
  }
}


.animate-choose-list-wrapper {
  position: fixed;
  top: 0;
  right: 0;
  width: 380px;
  height: calc(100% - 48px);
  background: white;
  z-index: 100;
  transition: all 0.28s;
  &.fadeInUp {
    top: 50px;
    opacity: 1;
  }
  &.fadeInDown {
    opacity: 0;
    top: 110%;
  }
}

.animate-choose-item {
  height: 100%;
  .animate-choose-item-inner {
    display: inline-block;
    width: 25%;
    height: 72px;
    color: rgb(21, 147, 255);
    text-align: center;
    cursor: pointer;
    & > .animate-choose-image {
      display: inline-block;
      width: 40px;
      height: 40px;
      margin-bottom: 6px;
      background-image: url(../../../../common/images/use-beb469.png);
      background-position:0 -480px;
    }
    p {
      font-size: 12px;
      line-height: 1;
    }
  }
}
.animate-choose-item .unclick {
  // border: #666 solid 1px;
  border-radius: 20%;
  color: rgb(147, 142, 142);
  & > .animate-choose-image {
      display: inline-block;
      width: 40px;
      height: 40px;
      margin-bottom: 6px;
      background-image: url(../../../../common/images/use-beb469.png);
      background-position:0 0;
  }
}

.el-animate-list-wrapper {
  .el-animate-title-name {
    display: inline-block;
    vertical-align: middle;
    font-size: 14px;
    font-weight: bold;
    width: 55px;
  }
  .el-animate-title-type-wrapper {
    display: inline-block;
    width: 164px;
  }
  .el-animate-title-type {
    display: inline-block;
    vertical-align: middle;
    max-width: 140px;
    height: 24px;
    line-height: 1px;
    color: #333;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    background: #fafafa;
    border-radius: 60px;
    padding: 4px 16px;
    border: none;
    cursor: pointer;
    margin: 0 20px 0 10px;
    &:hover {
      color: white;
      background: $primary;
    }
  }
  .el-animate-title-btn {
    display: inline-block;
    vertical-align: middle;
    text-align: center;
    line-height: 24px;
    font-size: 14px;
    color: #666;
    width: 24px;
    height: 24px;
    cursor: pointer;
    border-radius: 4px;
    background: rgba(37, 165, 137, 0.08);
    margin-left: 6px;
    &:hover {
      color: white;
      background: $primary;
    }
  }
}
//随机按钮的样式
.random-animate-button {
  display: table;
  margin: 0 auto;
  color:#fff;
  background: rgb(153, 153, 153);
}
</style>

<style lang="scss">
.components-attr-edit {
  .el-tabs {
    height: 100%;
    padding-left: 0px;
    padding-right: 0px;
    padding-bottom: 0px;
  }
}

.animate-choose-list-wrapper {
}
</style>
