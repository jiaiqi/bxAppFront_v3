<template>
  <view class="selector-wrap cu-timeline">
    <view class="parent-node cu-item text-green animation-slide-top">
      <view class="parent-list  ">
        <view class="list-item" v-for="(item, index) in treeData" :key="index" @click="clickParentNode(item, index)">
          <view class="label" :class="item[nodeKey] === activeNode[nodeKey] ? 'active' : 'normal'">{{ item[disColName] }}</view>
        </view>
      </view>
    </view>
    <treeSelector
      :key="key"
      v-if="childNode.length > 0"
      :name="activeNode[nodeKey]"
      :treeData="childNode"
      :childNodeCol="childNodeCol"
      :disColName="disColName"
      @clickLastNode="clickLastNode"
      :nodeKey="nodeKey"
    ></treeSelector>
  </view>
</template>

<script>
export default {
  name: 'treeSelector',
  data() {
    return {
      activeNode: {},
      childNode: [],
      key: false
    };
  },
  props: {
    treeData: {
      type: Array,
      default: () => {}
    },
    childNodeCol: {
      type: String,
      default: '_childNode'
    },
    disColName: {
      type: String,
      default: 'name'
    },
    nodeKey: {
      type: String,
      default: 'no'
    }
  },
  methods: {
    onTree(e) {
      if (e.item._childNode.length === 0) {
        this.$emit('clickLastNode', e);
      }
    },
    clickParentNode(item, index) {
      this.key = !this.key;
      if (item._childNode.length === 0) {
        this.$emit('clickLastNode', { item, index });
      } else {
        if (this.activeNode[this.nodeKey] === item[this.nodeKey]) {
          this.activeNode = {};
          this.childNode = [];
          debugger;
        } else {
          this.activeNode = item;
          this.childNode = item[this.childNodeCol];
        }
      }
      this.$emit('clickParentNode', { item, index });
    },
    clickLastNode(e) {
      //点击最底层节点
      if (e.item._childNode.length === 0) {
        this.$emit('clickLastNode', e);
      }
    }
  },
  treeData: {
    handler: function(nVal, oVal) {
      console.log('newVal', nVal);
      this.childNode.length = 0;
      this.activeNode = {};
    },
    deep: true
  }
};
</script>

<style scoped lang="scss">
.cu-timeline > .cu-item::before {
  background-color: #efefef;
}
.cu-timeline > .cu-item::after {
  background-color: #39b54a;
  // background-color: #999;
}
.cu-timeline > .cu-item {
  padding: 5px 15px 5px 60px;
}
.cu-timeline {
  background-color: #efefef;
}
.left-timeline {
  &::before {
    font-family: 'cuIcon';
    display: block;
    position: absolute;
    top: 18px;
    z-index: 9;
    background-color: #ffffff;
    width: 25px;
    height: 25px;
    text-align: center;
    border: none;
    line-height: 25px;
    left: 18px;
    content: '⚪';
  }
  &::after {
    content: '';
    display: block;
    position: absolute;
    width: 0.5px;
    background-color: #ddd;
    left: 30px;
    height: 50%;
    top: 0;
    z-index: 8;
  }
}

.selector-wrap {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  width: 100%;
  .parent-node {
    display: flex;
    width: 100%;
    .line-box {
      width: 30upx;
      height: 100upx;
      flex: 1;
    }
  }
  .parent-list {
    width: 100%;
    background-color: #fff;
    min-height: 200upx;
    border-radius: 20upx;
    display: flex;
    flex-wrap: wrap;
    align-items: flex-start;
    color: #333;
    box-shadow: 3px 3px 5px rgba($color: #333, $alpha: 0.5);
    .list-item {
      box-sizing: border-box;
      .label {
        padding: 10upx 20upx;
        margin: 10upx;
        // border: 1px solid #0081ff;#39b54a
        border: 1px solid #39b54a;
        border-radius: 20upx;
        &.active {
          background-color: #39b54a;
          // background-color: #0081ff;
          color: #fff;
        }
        &.normal {
          box-sizing: border-box;
        }
      }
    }
  }
}
[class*='animation-'] {
  animation-duration: 0.5s;
  animation-timing-function: ease-out;
  animation-fill-mode: both;
}
.animation-slide-top {
  animation-name: slide-top;
}
.animation-slide-bottom {
  animation-name: slide-top;
  animation-direction: reverse;
}
@keyframes slide-top {
  0% {
    opacity: 0;
    transform: translateY(-100%);
  }
  50%{
    opacity: .1;
    transform: translateY(-50%);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
