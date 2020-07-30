<template>
  <swiper class="screen-swiper round-dot" :indicator-dots="true" :circular="true" :autoplay="false" interval="5000" duration="500">
    <swiper-item v-for="(item, index) in menuList" :key="index">
      <view class="bg-white  menu-item">
        <view class="bg-imgs " v-for="(item2, index2) in item" :key="index2" @click="clickMenu(item2)">
          <view class="menu-pic bg-blue">{{ item2[menuTemplate['lableCol']].slice(0, 2) }}</view>
          <!-- <view class="menu-pic bg-blue" v-if="!item2.type">{{ item2.dest_app }}</view> -->
          <!-- <view class="menu-pic2" v-if="item2.type && item2.type === 'more'"><image src="../../../static/img/more2.png" mode=""></image></view> -->
          <view class="label">{{ item2[menuTemplate['lableCol']] }}</view>
        </view>
      </view>
    </swiper-item>
  </swiper>
</template>

<script>
export default {
  data() {
    return {
      // menuList:[],
    };
  },
  props: {
    menuList: {
      type: Array,
      default: () => {
        [];
      }
    },
    menuTemplate: {
      type: Object,
      default: () => {
        return {
          lableCol: 'dest_menu_no'
        };
      }
    }
  },
  methods: {
    clickMenu(e) {
      this.$emit('clickMenu', e);
    }
  }
};
</script>

<style scoped lang="scss">
.screen-swiper {
  margin: 20upx auto;
  background-color: #fff;
  // border-radius: 20upx;
  overflow: hidden;
  width: 750upx;
  // margin: 0 auto;
}
// @media screen and (min-width: 750px){
//   .screen-swiper {
//     max-width: 500px;
//     max-height: 370px;
//   }
// }
.menu-item {
  display: flex;
  box-sizing: border-box;
  flex-wrap: wrap;
  // justify-content: space-around;
  .bg-imgs {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin: 10upx;
    width: 22%;
    max-width: 100px;
    .menu-pic2 {
      width: 100upx;
      height: 100upx;
      border-radius: 20upx;
    }
    .menu-pic {
      width: 100upx;
      height: 100upx;
      border-radius: 20upx;
      font-size: 30upx;
      margin: 0 auto;
      justify-content: center;
      align-items: center;
      display: flex;
      font-family: '黑体';
      // background-color: #409EFF;
      z-index: 1;
      &::before {
        content: '';
        position: absolute;
        width: 100upx;
        height: 100upx;
        // opacity: 1;
        background: rgba($color: #0081ff, $alpha: 0.8);
        transform: translate(3px, 4px);
        border-radius: 20upx;
        z-index: -1;
        box-shadow: 10px 10px 22px 2px rgba(0, 0, 100, 0.2);
      }
      &:active {
        transform: translate(2px, 2px);
      }
    }
    .label {
      line-height: 50upx;
      margin-top: 10upx;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      // font-size: .16rem;
    }
  }
}
</style>
