<template>
  <div>
    <el-button type="primary" class="btn-style" @click="saveData">保存</el-button>
    <div class="approval-flow fd-nav-content">
      <div class="dingflow-design">
        <div class="zoom flex">
          <div :class="'zoom-out'+ (nowVal==50?' disabled':'')" @click="zoomSize(1)"></div>
          <span>{{nowVal}}%</span>
          <div :class="'zoom-in'+ (nowVal==300?' disabled':'')" @click="zoomSize(2)"></div>
        </div>
        <div class="box-scale" id="box-scale" :style="'transform: scale('+nowVal/100+'); transform-origin: 50% 0px 0px;'">
          <div class="end-node flex">
              <div class="title">开始表单</div>
              <div class="content">选择表单</div>
          </div>
          <nodeWrap v-bind="$attrs" :nodeConfig="nodeConfig" :isTried="isTried"></nodeWrap>
          <div class="end-node flex">
            <div class="title">结束表单</div>
            <div class="content">选择表单</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import nodeWrap from './child/nodeWrap.vue'
export default {
  components: {
    nodeWrap,
  },
  data() {
    return {
      //流程设置
      isTried: false,
      tipList: [],
      nodeConfig: {
        error: true,
        childNode: {
          nodeName: "运算表单",
          nodeUserType: {
            valueName: "admin",
            valueList: ["admin"],
            type: "user",
            value: "admin"
          },
          examineMode: "1",
          error: false,
          type: 1,
          nodeId: "approvalID"
        }
      },
      nowVal: 100,
      flowPermission: [],
      tableId: "",
      nodeConfigss: {
        "type": 0, //类型
        "priorityLevel": "", //优先级
        "settype": "", //审批人类型
        "selectMode": "", // 1人还是多人
        "selectRange": "", //选择范围
        "directorLevel": "", // 主管层级
        "examineMode": "", //多人审批时采用的审批方式  （依次审批 ，会签）
        "noHanderAction": "", //审批人为空时 (自动审批通过/不允许发起, 转交给审核管理员)
        "examineEndType": "",
        "examineEndRoleId": "",
        "examineEndDirectorLevel": "", // 审批终点层级
        "ccSelfSelectFlag": "", // 抄送人
        "conditionList": [], //条件设置
        "nodeUserList": [], // 人员列表
      },
    };
  },
  computed: {

  },
  watch: {
  },
  created() {
  console.log(this.nodeConfig,this.dataFields,this.isTried,'dasd')
  },
  mounted() {

  },
  methods: {
    // 保存
    saveData() {
      this.isTried = true;
      this.tipList = [];
      this.reErr(this.nodeConfig)
      if (this.tipList.length != 0) {
        return;
      }
      this.workflowSave()
    },
    reErr(data) {
      if (data.childNode) {
        if (data.childNode.type == 1) {
          //审批人
          if (data.childNode.error) {
            this.tipList.push({
              name: data.childNode.nodeName,
              type: '审核人',
            })
          }
          this.reErr(data.childNode)
        } else if (data.childNode.type == 2) {
          if (data.childNode.error) {
            this.tipList.push({
              name: data.childNode.nodeName,
              type: '抄送人',
            })
          }
          this.reErr(data.childNode)
        } else if (data.childNode.type == 3) {
          this.reErr(data.childNode.childNode)
        } else if (data.childNode.type == 4) {
          this.reErr(data.childNode)
          for (
            var i = 0; i < data.childNode.conditionNodes.length; i++
          ) {
            if (data.childNode.conditionNodes[i].error) {
              this.tipList.push({
                name: data.childNode.conditionNodes[i].nodeName,
                type: '条件',
              })
            }
            this.reErr(data.childNode.conditionNodes[i])
          }
        }
      } else {
        data.childNode = null
      }
    },
    // 给数据加preId(递归)
    handleData(data) {
      if (data.childNode) {
        data.childNode.preId = data.nodeId;
        this.handleData(data.childNode);
      }
      return data;
    },
    workflowSave() {
      // 调接口存数据
      let data = this.handleData(this.nodeConfig);
      console.log('data', data);
      this.$message.success('保存成功')
    },
    zoomSize(type) {
      if (type == 1) {
        if (this.nowVal == 50) {
          return;
        }
        this.nowVal -= 10;
      } else {
        if (this.nowVal == 300) {
          return;
        }
        this.nowVal += 10;
      }
    }
  },
};
</script>

<style lang="scss">
.btn-style {
  position: fixed;
  right: 60px;
}
.zoom {
  position: fixed;
  height: 40px;
  width: 125px;
  right: 40px;
  margin-top: 30px;
  z-index: 10;
  display: flex;
}
.zoom-in,
.zoom-out {
  width: 30px;
  height: 30px;
  background: #fff;
  color: #c1c1cd;
  cursor: pointer;
  background-size: 100%;
  background-repeat: no-repeat;
}

.zoom-out {
  background-image: url(https://gw.alicdn.com/tfs/TB1s0qhBHGYBuNjy0FoXXciBFXa-90-90.png);
}

.zoom-out.disabled {
  opacity: 0.5;
}

.zoom-in {
  background-image: url(https://gw.alicdn.com/tfs/TB1UIgJBTtYBeNjy1XdXXXXyVXa-90-90.png);
}

.zoom-in.disabled {
  opacity: 0.5;
}
.box-scale {
  transform: scale(1);
  display: inline-block;
  position: relative;
  width: 100%;
  // padding: 54.5px 0;
  transform-origin: 0 0 0;
  left: 40%;
}
.branch-wrap .branch-box-wrap{
  margin-left: -240px;

}
.branch-wrap .branch-box-wrap .auto-judge {
   position: relative;
   width: 280px;
   min-height: 100px;
   background: #fff;
   border-radius: 4px;
   cursor: pointer;

 }
.branch-wrap .branch-box-wrap .auto-judge .title-wrapper {
  position: relative;
  font-size: 12px;
  color: #fff;
  text-align: center;
  height: 38px;
  line-height: 38px;
  padding: 0 13px 0 0px;
  background: #c1c1cd;
  border-bottom: 1px solid #ebebeb;
}
.node-wrap-box {
  display: inline-flex;
  flex-direction: column;
  position: relative;
  width: 280px;
  min-height: 72px;
  background: #fff;
  border-radius: 4px;
  cursor: pointer;
}
.node-wrap-box:after {
  pointer-events: none;
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 2;
  border-radius: 4px;
  border: 1px solid transparent;
  transition: all 0.1s cubic-bezier(0.645, 0.045, 0.355, 1);
  box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.1);
}
.end-node {
  display: inline-flex;
  flex-direction: column;
  position: relative;
  width: 280px;
  min-height: 72px;
  background: #fff;
  border-radius: 4px;
  cursor: pointer;

}
.end-node .title {
  position: relative;
  display: flex;
  align-items: center;
  padding-left: 16px;
  height: 40px;
  line-height: 40px;
  font-size: 16px;
  color: #fff;
  font-size: 14px;
  padding-left: 18px;
  text-align: left;
  background: #576a95;
  border-radius: 4px 4px 0 0;
}
.end-node .content {
  position: relative;
  font-size: 14px;
  padding: 20px 18px;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}
.fd-nav-content {
  position: fixed;
  top: 60px;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
  overflow-x: hidden;
  overflow-y: auto;
  padding-bottom: 30px;
}
.dingflow-design {
  width: 100%;
  background-color: #f5f8ff;
  overflow: auto;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  top: 0;
}
.editable-title {
  font-size: 14px;
  padding-left: 18px;
}
.dingflow-design .ie-polyfill-container {
  display: -ms-grid;
  -ms-grid-columns: min-content;
}

.dingflow-design .node-wrap {
  flex-direction: column;
  -webkit-box-pack: start;
  -ms-flex-pack: start;
  justify-content: flex-start;
  -webkit-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
  -webkit-box-flex: 1;
  -ms-flex-positive: 1;
  // padding: 0 50px;
  position: relative;
}

.dingflow-design .branch-wrap,
.dingflow-design .node-wrap {
  display: inline-flex;
  // width: 100%;
}

.dingflow-design .branch-box-wrap {
  display: flex;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  -ms-flex-direction: column;
  flex-direction: column;
  -ms-flex-wrap: wrap;
  flex-wrap: wrap;
  -webkit-box-align: center;
  -ms-flex-align: center;
  align-items: center;
  min-height: 270px;
  width: 100%;
  -ms-flex-negative: 0;
  flex-shrink: 0;
}

.dingflow-design .branch-box {
  display: flex;
  overflow: visible;
  min-height: 180px;
  height: auto;
  border-bottom: 1px solid #ebebeb;
  border-top: 1px solid #ebebeb;
  position: relative;
  margin-top: 15px;
}

.dingflow-design .branch-box .col-box:before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 0;
  margin: auto;
  width: 1px;
  height: 100%;
  background-color: #ebebeb;
}

.dingflow-design .add-branch {
  border: none;
  outline: none;
  user-select: none;
  justify-content: center;
  font-size: 12px;
  padding: 0 10px;
  height: 30px;
  line-height: 30px;
  border-radius: 15px;
  color: #4880ff;
  background: #fff;
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
  position: absolute;
  top: -16px;
  left: 50%;
  transform: translateX(-50%);
  transform-origin: center center;
  cursor: pointer;
  z-index: 1;
  display: inline-flex;
  align-items: center;
  -webkit-transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
  transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
}

.dingflow-design .add-branch:hover {
  transform: translateX(-50%) scale(1.1);
  box-shadow: 0 8px 16px 0 rgba(0, 0, 0, 0.1);
}

.dingflow-design .add-branch:active {
  transform: translateX(-50%);
  box-shadow: none;
}

.dingflow-design .col-box {
  display: inline-flex;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  flex-direction: column;
  -webkit-box-align: center;
  align-items: center;
  position: relative;
}

.dingflow-design .condition-node {
  min-height: 220px;
}

.dingflow-design .condition-node,
.dingflow-design .condition-node-box {
  display: inline-flex;
  -webkit-box-orient: vertical;
  -webkit-box-direction: normal;
  flex-direction: column;
  -webkit-box-flex: 1;
}

.dingflow-design .condition-node-box {
  padding-top: 30px;
  padding-right: 50px;
  padding-left: 50px;
  -webkit-box-pack: center;
  justify-content: center;
  -webkit-box-align: center;
  align-items: center;
  flex-grow: 1;
  position: relative;

}

.dingflow-design .condition-node-box:before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  width: 1px;
  height: 100%;
  background-color: #ebebeb;
}

.dingflow-design .auto-judge:after {
  pointer-events: none;
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 2;
  border-radius: 4px;
  border: 1px solid transparent;
  transition: all 0.1s cubic-bezier(0.645, 0.045, 0.355, 1);
  box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.1);
}

.dingflow-design .auto-judge.active:after,
.dingflow-design .auto-judge:active:after,
.dingflow-design .auto-judge:hover:after {
  border: 1px solid #4880ff;
  box-shadow: 0 0 6px 0 rgba(50, 150, 250, 0.3);
}

.dingflow-design .auto-judge.active .close,
.dingflow-design .auto-judge:active .close,
.dingflow-design .auto-judge:hover .close {
  display: block;
}


.dingflow-design .auto-judge .close {
  display: none;
  position: absolute;
  right: 22px;
  top: 8px;
  width: 30px;
  height: 30px;
  font-size: 14px;
  color:#ffffff;
  border-radius: 50%;
  text-align: center;
  line-height: 20px;
  z-index: 2;
}

.dingflow-design .auto-judge .content {
  height: calc(100% - 39px);
  display: flex;
  align-items: center;
  max-width: calc(100% - 36px);
  padding: 0 18px;
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 14px;
}


.dingflow-design .add-node-btn {
  user-select: none;
  width: 240px;
  padding: 30px 0;
  display: flex;
  -webkit-box-pack: center;
  justify-content: center;
  flex-shrink: 0;
  -webkit-box-flex: 1;
  flex-grow: 1;
  // margin-left: -55px;
}

.dingflow-design .add-node-btn .btn {
  outline: none;
  display: inline-block;
  width: 120px;
  height: 30px;
  background: #4880ff;
  text-align: center;
  position: relative;
  border: none;
  line-height: 30px;
  color: #ffffff;
  font-size: 14px;
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
  border-radius:10px;
  -webkit-transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
  transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
}

.dingflow-design .add-node-btn .btn .iconfont {
  color: #fff;
  font-size: 16px;
}

.dingflow-design .add-node-btn .btn:hover {
  transform: scale(1.3);
  box-shadow: 0 13px 27px 0 rgba(0, 0, 0, 0.1);
}

.dingflow-design .add-node-btn .btn:active {
  transform: none;
  background: #1e83e9;
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
}

.top-left-cover-line {
  left: -1px;
}

.top-left-cover-line,
.top-right-cover-line {
  position: absolute;
  height: 3px;
  width: 50%;
  background-color: #f5f8ff;
  top: -2px;
}

.top-right-cover-line {
  right: -1px;
}

.bottom-left-cover-line {
  left: -1px;
}

.bottom-left-cover-line,
.bottom-right-cover-line {
  position: absolute;
  height: 3px;
  width: 50%;
  background-color: #f5f8ff;
  bottom: -2px;
}

.bottom-right-cover-line {
  right: -1px;
}

.node-wrap-box.active:after,
.node-wrap-box:active:after,
.node-wrap-box:hover:after {
  border: 1px solid #3296fa;
  box-shadow: 0 0 6px 0 rgba(50, 150, 250, 0.3);
}

.node-wrap-box.active .close,
.node-wrap-box:active .close,
.node-wrap-box:hover .close {
  display: block;
}

.node-wrap-box.error:after {
  border: 1px solid #f25643;
  box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.1);
}

.node-wrap-box .title {
  position: relative;
  display: flex;
  align-items: center;
  padding-left: 16px;
  height: 40px;
  line-height: 40px;
  font-size: 16px;
  color: #fff;
  text-align: center;
  background: #c1c1cd;

}

.node-wrap-box .title .iconfont {
  font-size: 12px;
  margin-right: 5px;
}

.node-wrap-box .placeholder {
  color: #bfbfbf;
}

.node-wrap-box .close {
  display: none;
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  height: 20px;
  font-size: 14px;
  color: #fff;
  border-radius: 50%;
  text-align: center;
  line-height: 20px;
}

.node-wrap-box .content {
  position: relative;
  font-size: 14px;
  padding: 20px 18px;
}

.node-wrap-box .content .text {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}

.node-wrap-box .content .arrow {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  height: 14px;
  font-size: 14px;
  color: #979797;
}

.start-node.node-wrap-box .content .text {
  display: block;
  white-space: nowrap;
}
</style>
