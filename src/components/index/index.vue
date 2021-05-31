	<template>
  <div class="container">
    <el-menu class="topNav" mode="horizontal" background-color="#545c64" text-color="#fff" active-text-color="#ffd04b">
      <el-menu-item index="1">编辑</el-menu-item>
      <el-menu-item index="2">预览</el-menu-item>
      <el-menu-item index="3">清空</el-menu-item>
      <el-menu-item index="4">新增页面</el-menu-item>
      <el-menu-item index="5">问题答疑</el-menu-item>
      <el-menu-item index="6">注册/登录</el-menu-item>
    </el-menu>
    <el-row class="rowBox">
      <el-col :span="3" class="rowBoxLeft">
        <!-- 布局 -->
        <draggable
          class="layoutBox"
          :group="{name:'layout',put:false}"
          v-model="layout"
          :options="dragOption1"
          @end="endLayout"
        >
          <div class="columns" v-for="(item,index) in layout" :key="index">
            <div
              class="column"
              v-for="(item2,index2) in item.columns"
              :key="index2"
            >{{ item2.value }}</div>
          </div>
        </draggable>
        <!-- 组件 -->
        <draggable
          class="componentsBox"
          :group="{name:'components',put:false}"
          v-model="components"
          :options="dragOption1"
          @end="endComponent"
        >
          <div class="columns" v-for="(item,index) in components" :key="index">
            <MyComponent :html="item.html"></MyComponent>
          </div>
        </draggable>
      </el-col>
      <el-col :span="21" class="rowBoxMiddle">
        <!-- 目标 -->
        <draggable
          class="webBox"
          group="layout"
          v-model="listData"
          :options="dragOption2"
          @update="updateColumns"
        >
          <div
            class="columns"
            v-for="(item,index) in listData"
            :key="index"
            @mouseenter="hoverColumns(item,'enter')"
            @mouseleave="hoverColumns(item,'leave')"
          >
            <div class="column" v-for="(item2,index2) in item.columns" :key="index2">
              <draggable
                class="componentBox"
                group="components"
                v-model="item2.component"
                :options="dragOption2"
              >
                <div v-for="(item3,index3) in item2.component" :key="index3">
                  <MyComponent :html="item3.html"></MyComponent>
                </div>
              </draggable>
            </div>
            <div class="operationBox" v-show="item.operationstaus === true">
              <el-button size="mini" @click="deleteColumns(index)">删除</el-button>
              <el-button size="mini" type="primary" @click="editColumns(item,index)">编辑</el-button>
            </div>
          </div>
        </draggable>
      </el-col>
    </el-row>

    <el-drawer
      custom-class="rowBoxRight"
      title="我是标题"
      :visible.sync="rowBoxRight"
      size="300px"
      :with-header="false"
    >
      <div class="layoutConfig" v-show="listData.length > 0">
        <el-input v-model="layoutConfig.column" @input="layoutConfigColumn"></el-input>
      </div>
      <div class="componentConfig"></div>
    </el-drawer>
  </div>
</template>

	<script>
import Vue from "vue";
import draggable from "vuedraggable";
export default {
  name: "hello",
  components: {
    draggable,
    MyComponent: {
      props: {
        html: String
      },
      render(h) {
        const com = Vue.extend({
          template: this.html
        });

        return h(com, {});
      }
    }
  },
  data() {
    return {
      layout: [],
      components: [],
      listData: [],
      dragOption1: {
        animation: 300,
        sort: false
      },
      dragOption2: {
        animation: 300
      },
      layoutConfig: {
        currentItem: Object,
        currentIndex: Number,
        column: "1"
      },
      rowBoxRight: false
    };
  },
  created() {
    this.getLayout();
    this.getComponents();
  },
  methods: {
    getLayout() {
      this.layout = [
        {
          name: "1",
          operationstaus: false,
          columns: [
            {
              value: "一列",
              component: []
            }
          ]
        }
      ];
    },
    getComponents() {
      this.components = [
        {
          html: '<el-button type="primary">按钮</el-button>'
        },
        {
          html: "<el-input></el-input>"
        },
        {
          html:
            '<el-card class="box-card"><div slot="header" class="clearfix"><span>卡片名称</span><el-button style="float: right; padding: 3px 0" type="text">操作按钮</el-button></div><div v-for="o in 4" :key="o" class="text item">列表内容{{o}}</div></el-card>'
        }
      ];
    },
    endLayout(e) {
      if (e.pullMode === undefined) {
        this.$message({
          showClose: true,
          message: "请将布局移动到正确位置",
          type: "warning"
        });
      } else {
        this.getLayout();
      }
    },
    endComponent(e) {
      if (e.pullMode === undefined) {
        this.$message({
          showClose: true,
          message: "请将组件移动到布局中",
          type: "warning"
        });
      } else {
        this.getComponents();
      }
    },

    //选中布局
    hoverColumns(item, type) {
      if (type === "enter") {
        item.operationstaus = true;
      } else if (type === "leave") {
        item.operationstaus = false;
      } else {
        item.operationstaus = false;
      }
    },

    //编辑布局
    editColumns(item, index) {
      this.layoutConfig.column = item.name;
      this.layoutConfig.currentIndex = index;
      this.layoutConfig.currentItem = item;
      this.rowBoxRight = true;
    },

    //删除布局
    deleteColumns(index) {
      this.$confirm("是否删除该区域布局?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.listData.splice(index, 1);
        })
        .catch(() => {});
    },

    //更新布局顺序，重新选中布局
    updateColumns(e) {
      console.log(e);
      this.layoutConfig.currentIndex = e.newIndex;
    },

    //布局配置（改变列数）
    layoutConfigColumn() {
      this.listData[
        this.layoutConfig.currentIndex
      ].name = this.layoutConfig.column;
      let arr = [];
      for (let i = 0; i < this.layoutConfig.column; i++) {
        let obj = {
          value: "一列",
          component: []
        };
        arr.push(obj);
      }
      this.listData[this.layoutConfig.currentIndex].columns = arr;
    }
  }
};
</script>

<style lang="less">
.container {
  height: 100%;
  .topNav {
    border: none;
    height: 60px;
  }
  .rowBox {
    height: calc(100% - 60px);
    position: relative;
    .rowBoxLeft {
      min-height: 100%;
      padding: 20px;
      box-sizing: border-box;
      max-height: 100%;
      overflow: auto;
      background-color: #efefef;
      position: relative;
      z-index: 1;
      .layoutBox {
        .columns {
          display: flex;
          margin: 10px;
          .column {
            flex: 1;
            min-height: 50px;
            line-height: 50px;
            text-align: center;
            border: 1px dashed #ccc;
            background-color: #fff;
          }
        }
      }
    }
    .rowBoxMiddle {
      height: 100%;
      background-color: #fff;
      position: relative;
      z-index: 1;
      .webBox {
        min-height: 100%;
        padding: 20px;
        box-sizing: border-box;
        max-height: 100%;
        overflow: auto;
        .columns {
          display: flex;
          position: relative;
          .column {
            flex: 1;
            border: 1px dashed #ccc;
            background-color: #fff;
            padding: 25px;
            box-sizing: border-box;
            .componentBox {
              height: 100%;
            }
          }
          .operationBox {
            position: absolute;
            right: 0;
            top: 0;
          }
        }
      }
    }
  }
  .rowBoxRight {
    padding: 20px;
    box-sizing: border-box;
  }
}
</style>