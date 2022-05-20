<template>
  <div class="add-node-btn-box">
    <div class="add-node-btn">
      <el-popover placement="right-start" popper-class="add-node-popover" v-model="visible">
        <div class="add-node-popover-body">
          <div class="add-node-popover-item approver" @click="addType(1)">
            单分支
          </div>
          <div class="add-node-popover-item notifier" @click="addType(4)">
            多分支
          </div>
        </div>
        <span class="btn" slot="reference">添加表单</span>
      </el-popover>
    </div>
  </div>
</template>
<script>
export default {
  props: ["childNodeP", "nodeConfig", 'tip'],
  data() {
    return {
      visible: false,
      parentObj: {},
    }
  },
  methods: {
    addType(type) {
      this.visible = false;
      let data;
      if (type != 4) {
        if (type == 1) {
          data = {
            nodeName: "运算表单",
            error: true,
            type: 1,
            nodeId: "approvalID",
            examineMode: "1",
            nodeUserType: {
              type: 'manager',
              value: '',
              valueList: [],
              valueName: '',
            },
            childNode: this.childNodeP,
          }
        }
      } else {
        data = {
          nodeName: "路由",
          type: 4,
          nodeId: "conditionID",
          childNode: this.childNodeP,
          conditionNodes: [{
            nodeName: "条件1",
            error: true,
            type: 3,
            priorityLevel: 1,
            conditionList: [],
            childNode: null,
          }, {
            nodeName: "默认",
            error: true,
            type: 3,
            priorityLevel: 2,
            conditionList: [],
            childNode: null
          }]
        }
      }
      this.$emit("update:childNodeP", data);
    }
  }
}
</script>
<style lang="scss" scoped>
.add-node-btn-box {
  width: 240px;
  display: inline-flex;
  flex-shrink: 0;
  position: relative;
  &:first-child {
    margin-left: 16px;
  }
  &:before {
    content: "";
    position: absolute;
    top: 1px;
    left: 0px;
    right: 0;
    bottom: 0;
    z-index: -1;
    margin: auto;
    width: 1px;
    // height: 100%;
    background-color: #ebebeb;
  }
}
.img-style {
  width: 36px;
}
.add-node-popover {
  padding: 14px 26px;
  .add-node-popover-body {
    display: flex;
    .add-node-popover-item {
      width: 70px;
      height: 30px;
      line-height: 30px;
      background: #4880ff;
      font-size: 12px;
      border-radius: 20px;
      color: #ffffff;
      margin-right: 20px;
      text-align: center;
    }
  }
}
</style>
