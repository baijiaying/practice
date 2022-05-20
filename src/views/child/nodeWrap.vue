<template>
  <div class="nodeflow-components">
    <div class="node-wrap" v-if="nodeConfig.type != 4">
      <div class="node-wrap-box" v-if="nodeConfig.type && nodeConfig.type != 0" >
        <div class="title">
          <span class="editable-title">{{ nodeConfig.nodeName }}</span>
          <i class="el-icon-close close" v-if="nodeConfig.type != 0" @click.stop="delNode()"></i>
        </div>
        <div class="content">
          {{ setApproverStr(nodeConfig) }}
        </div>
      </div>
      <addNode :childNodeP.sync="nodeConfig.childNode"></addNode>
    </div>
    <div class="branch-wrap" v-if="nodeConfig.type == 4">
      <div class="branch-box-wrap">
        <div class="branch-box">
          <button class="add-branch" @click="addTerm">
            添加条件
          </button>
          <div class="col-box" v-for="(item, index) in nodeConfig.conditionNodes" :key="index">
            <div class="condition-node">
              <div class="condition-node-box">
                <div class="auto-judge" :class="isTried && item.error ? 'error active' : '' ">
                  <div class="title-wrapper">
                    <span class="editable-title">运算表单</span>
                    <i class="el-icon-close close" @click.stop="delTerm(index)"></i>
                  </div>
                  <div class="content">
                    {{ conditionStr(item, index) }}
                  </div>
                </div>
                <addNode :childNodeP.sync="item.childNode" :tip="'条件'"></addNode>
              </div>
            </div>
            <nodeWrap v-if="item.childNode && item.childNode" :nodeConfig.sync="item.childNode" :isTried.sync="isTried"></nodeWrap>
            <div class="top-left-cover-line" v-if="index == 0"></div>
            <div class="bottom-left-cover-line" v-if="index == 0"></div>
            <div class="top-right-cover-line" v-if="index == nodeConfig.conditionNodes.length - 1"></div>
            <div class="bottom-right-cover-line" v-if="index == nodeConfig.conditionNodes.length - 1"></div>
          </div>
        </div>
        <addNode :childNodeP.sync="nodeConfig.childNode"></addNode>
      </div>
    </div>
    <nodeWrap v-if="nodeConfig.childNode && nodeConfig.childNode" :nodeConfig.sync="nodeConfig.childNode" :isTried.sync="isTried"></nodeWrap>
  </div>
</template>
<script>
import addNode from './addNode'

export default {
  name: 'nodeWrap',
  components: {addNode,},
  props: ['nodeConfig', 'isTried'],
  data() {
    return {
      //条件弹框字段
      conditionList: [],
    }
  },
  watch: {},
  computed: {},
  mounted() {
    if (this.nodeConfig.type == 1) {
      this.nodeConfig.error = this.setApproverStr(this.nodeConfig) == ''
    } else if (this.nodeConfig.type == 2) {
      this.nodeConfig.error = this.setApproverStr(this.nodeConfig) === ''
    } else if (this.nodeConfig.type == 4) {
      for (var i = 0; i < this.nodeConfig.conditionNodes.length; i++) {
        this.nodeConfig.conditionNodes[i].error =
            this.conditionStr(this.nodeConfig.conditionNodes[i], i) ==
            '请设置条件'
      }
    }
  },
  methods: {
    //条件显示
    conditionStr(item, index) {
      let {
        conditionList, //条件组
        conditionStringName, //条件显示
      } = item
      let arr = [true] //判断条件是否有值
      if (conditionList.length === 0) {
        return '请选择表单'
      }
      if (conditionList.length !== 0) {
        if (conditionList.length === 1) {
          //当条件组为一个
          conditionList[0].conditionChildrenNodes && conditionList[0].conditionChildrenNodes.forEach((item, index) => {
                if (item.leftFileds == '' || item.centerFileds == '' || item.rightFileds == '') {
                  arr.push(false)
                }
                if (index !== 0 && item.conditionOperator == '') {
                  arr.push(false)
                }
              }
          )
        } else {
          //当条件组为多个（判断是否有运算符）
          conditionList.forEach((item, index) => {
            if (index != conditionList.length - 1) {
              //条件组不为最后一个.校验是否有条件运算符
              if (item.conditionGroupOperator == '')
                arr.push(false)
            }
            if (item.conditionChildrenNodes && item.conditionChildrenNodes.length > 0) {
              item.conditionChildrenNodes.forEach((it, ind) => {
                if (it.leftFileds === '' || it.centerFileds === '' || it.rightFileds == '')
                  arr.push(false)
                if (ind !== 0 && it.conditionOperator == '')
                  arr.push(false)
              })
            } else if (item.conditionChildrenNodes && item.conditionChildrenNodes.length == 0) {
              arr.push(false)
            }
          })
        }
        if (arr.includes(false)) {
          return '请设置条件'
        }
      }
      return conditionStringName
    },
    //审批人抄送人显示和校验
    setApproverStr(nodeConfig) {
      return `运算表单`
    },
    //删除节点
    delNode() {
      this.$emit('update:nodeConfig', this.nodeConfig.childNode)
    },
    //添加条件
    addTerm() {
      let len = this.nodeConfig.conditionNodes.length;
      this.nodeConfig.conditionNodes.push({
        nodeName: '条件' + len,
        type: 3,
        priorityLevel: len + 1,
        conditionList: [],
        childNode: null,
      })
      for (var i = 0; i < this.nodeConfig.conditionNodes.length; i++) {
        this.nodeConfig.conditionNodes[i].error =
            this.conditionStr(this.nodeConfig.conditionNodes[i], i) ==
            '请设置条件' &&
            i != this.nodeConfig.conditionNodes.length - 1
      }
      this.$emit('update:nodeConfig', this.nodeConfig)
    },
    //删除条件
    delTerm(index) {
      this.nodeConfig.conditionNodes.splice(index, 1)
      for (var i = 0; i < this.nodeConfig.conditionNodes.length; i++) {
        this.nodeConfig.conditionNodes[i].error =
            this.conditionStr(this.nodeConfig.conditionNodes[i], i) ==
            '请设置条件' &&
            i != this.nodeConfig.conditionNodes.length - 1
      }
      this.$emit('update:nodeConfig', this.nodeConfig)
      if (this.nodeConfig.conditionNodes.length == 1) {
        if (this.nodeConfig.childNode) {
          if (this.nodeConfig.conditionNodes[0].childNode) {
            this.reData(this.nodeConfig.conditionNodes[0].childNode, this.nodeConfig.childNode)
          } else {
            this.nodeConfig.conditionNodes[0].childNode = this.nodeConfig.childNode
          }
        }
        this.$emit('update:nodeConfig', this.nodeConfig.conditionNodes[0].childNode)
      }
    },
    reData(data, addData) {
      if (!data.childNode) {
        data.childNode = addData
      } else {
        this.reData(data.childNode, addData)
      }
    },
    arrTransfer(index, type = 1) {
      //向左-1,向右1
      this.nodeConfig.conditionNodes[index] = this.nodeConfig.conditionNodes.splice(index + type, 1, this.nodeConfig.conditionNodes[index])[0];
      this.nodeConfig.conditionNodes.map((item, index) => {
        item.priorityLevel = index + 1
      })
      for (var i = 0; i < this.nodeConfig.conditionNodes.length; i++) {
        this.nodeConfig.conditionNodes[i].error =
            this.conditionStr(this.nodeConfig.conditionNodes[i], i) ==
            '请设置条件' &&
            i != this.nodeConfig.conditionNodes.length - 1
      }
      this.$emit('update:nodeConfig', this.nodeConfig)
    },
  },
}
</script>
<style lang="scss">
</style>
