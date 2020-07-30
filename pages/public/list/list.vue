<template>
  <view>
    <view class="search-bar" v-if="showSearchBar">
      <view class="cu-bar search bg-white fixed">
        <view class="search-form round">
          <text class="cuIcon-search"></text>
          <input @focus="searchBarFocus" @blur="serachBarBlur" :adjust-position="false" type="text" v-model="searchVal" placeholder="搜索图片、文章、视频" confirm-type="search" />
        </view>
        <view class="action"><button class="cu-btn bg-blue shadow-blur round" @click="toSearch">搜索</button></view>
      </view>
      <view style="height: 100upx;width: 100%;">占位</view>
    </view>
    <bx-list
      ref="bxList"
      :serviceName="serviceName"
      :condition="condition"
      :pageType="pageType"
      :viewTemp="viewTemp"
      :listConfig="listConfig"
      :searchWords="searchVal"
      :searchColumn="keyColumn"
      :rownumber="5"
      @click-list-item="clickItem"
      @list-change="listChange"
      @clickFootBtn="clickFootBtn"
      @loadEnd="loadEnd"
    ></bx-list>
    <view class="public-button-box"><view class="add-button" @click="clickAddButton" v-if="showAdd"></view></view>
  </view>
</template>

<script>
import bxList from '@/components/bx-list/bx-list.vue';
export default {
  components: { bxList },
  data() {
    return {
      serviceName: '',
      pageType: '',
      listConfig: {},
      srv_cols: [],
      condition: [],
      pattern: { color: '#7A7E83', backgroundColor: '#fff', selectedColor: '#007AFF', buttonColor: '#007AFF' },
      fabContent: [],
      fabHorizontal: 'left',
      fabVertical: 'bottom',
      fabDirection: 'horizontal',
      viewTemp: {
        // title: 'name',
        // tip: 'desc',
        // img: 'img',
        // price: 'current_price',
        // footer: 'shop_name'
      },
      publicButton: [],
      searchVal: '',
      keyColumn: 'name',
      showAdd: false,
      noData: false,
      showSearchBar: true
    };
  },
  onReachBottom() {
    console.log('触底');
    if (!this.noData) {
      uni.showLoading({
        title: '加载中',
        mask: true
      });
      this.$refs.bxList.pageInfo.pageNo++;
      this.$refs.bxList.getListData().then(data => {
        uni.hideLoading();
      });
    }
  },
  onLoad(option) {
    if (option.viewTemp) {
      let viewTemp = this.getDecodeUrl(option.viewTemp);
      this.viewTemp = JSON.parse(viewTemp);
    }
    if (option.hasOwnProperty('serviceName') && option.hasOwnProperty('pageType')) {
      console.log('list option:', option);
      this.serviceName = option.serviceName;
      this.pageType = option.pageType;
      this.getListV2();
    } else {
      // uni.showToast({
      // 	title:"无效页面",
      // 	icon:"Error"
      // })
      // setTimeout(()=>{
      // 	uni.hideToast()
      // 	uni.navigateBack()
      // },2000)
    }
  },
  methods: {
    toSearch() {
      let keywords = this.searchVal;
      this.$refs.bxList.toSearch();
    },
    loadEnd(e) {
      this.noData = true;
    },
    clickAddButton() {
      console.log('你点击了添加按钮:');
    },
    searchBarFocus(e) {
      console.log('searchBarFocus:', e);
    },
    serachBarBlur(e) {
      console.log('serachBarBlur:', e);
    },
    listChange(e) {
      console.log('listChange:', e);
    },
    clickItem(e) {
      let req = {
        button: {
          button_name: '详情',
          button_type: 'detail',
          service_name: this.serviceName
        },
        row: e
      };
      this.onButtonToUrl(req).then(res => {
        console.log('clickItem :', res);
      });
      console.log('click-list-item:', e);
    },
    clickFootBtn(data) {
      this.onButtonToUrl(data).then(res => {
        console.log('footBTN :', res);
        if (data.button && data.button.button_type === 'delete') {
          if (res.state === 'SUCCESS') {
            this.$refs.bxList.onRefresh();
          }
        }
      });
      console.log('clickFootBtn:', data);
    },
    async getListV2() {
      let app = uni.getStorageSync('activeApp');
      let colVs = await this.getServiceV2(this.serviceName, 'list', 'list', app);
      colVs.srv_cols = colVs.srv_cols.filter(item => item.in_list === 1);
      console.log('colVs', colVs);
      this.listConfig = colVs;
      this.publicButton = colVs.gridButton.filter(item => {
        if (item.permission === true) {
          switch (item.button_type) {
            case 'add':
              this.showAdd = true;
              return item;
              break;
            case 'select':
              this.showSearchBar = true;
              return item;
              break;
          }
        }
      });
      return colVs;
    }
  }
};
</script>

<style lang="scss" scoped>
.add-button {
  position: fixed;
  bottom: 100upx;
  left: calc(50% - 50upx);
  width: 100upx;
  height: 100upx;
  border-radius: 50%;
  background-color: #007aff;
  z-index: 100;
  &::before {
    position: absolute;
    width: 8upx;
    height: 60upx;
    left: calc(50% - 4upx);
    top: 20upx;
    content: '';
    background-color: #fff;
  }
  &::after {
    position: absolute;
    width: 60upx;
    height: 8upx;
    content: '';
    top: calc(50% - 4upx);
    left: 20upx;
    background-color: #fff;
  }
  &:active {
    transform: rotate(45deg);
    transition: all 0.2s;
  }
}
</style>
