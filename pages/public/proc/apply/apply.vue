<template>
  <view>
    <cu-custom bgColor="bg-blue" :isBack="true">
      <block slot="backText">返回</block>
      <block slot="content" v-if="colsV2Data.service_view_name">{{ colsV2Data.service_view_name }}</block>
      <!-- <block slot="right"><text class="cuIcon-add" style="font-size: 40upx;margin-right: 20upx;" @click="toApply"></text></block> -->
    </cu-custom>
    <view class="form-wrap" v-if="fields.length > 0">
      <bxform ref="bxForm" :pageType="type" :BxformType="type" :fields="fields"></bxform>
      <button type="primary" @click="submitData">提交</button>
    </view>
  </view>
</template>

<script>
import bxform from '@/components/bx-form/bx-form.vue';
export default {
  components: { bxform },
  data() {
    return {
      type: 'add',
      serviceName: '',
      fields: [],
      procBasicConfig: {},
      colsV2Data: {}
    };
  },
  methods: {
    async submitData() {
      let itemData = this.$refs.bxForm.getFieldModel();
      if (!itemData) {
        return;
      } else {
        let req = [
          {
            data: [itemData],
            serviceName: this.serviceName
          }
        ];
        console.log(req, 'req');
        let res = await this.onRequest('apply', this.serviceName, req, 'oa');
        if (res.data.state === 'SUCCESS') {
          console.log(res.data,'res.data');
          uni.showToast({
            title: res.data.resultMessage,
            icon: 'none'
          });
          uni.showModal({
            title:'提示',
            content: res.data.resultMessage+',即将返回流程列表',
            showCancel:false,
            success(res) {
              if(res.confirm){
                uni.redirectTo({
                  url:'../procList/procList'
                })
              }
            }
          })
         
        }
      }
    },
    async getBasicCfg() {
      // srvprocess_basic_cfg_select 流程初始化数据查询
      let serviceName = this.serviceName;
      let req = { serviceName: 'srvprocess_basic_cfg_select', colNames: ['*'], condition: [{ colName: 'service_name', ruleType: 'eq', value: serviceName }] };
      let res = await this.onRequest('select', 'srvprocess_basic_cfg_select', req, 'oa');
      if (res.data.state === 'SUCCESS') {
        this.procBasicConfig = res.data;
      }
    },
    async getColV2() {
      let colVs = await this.getServiceV2(this.serviceName, this.type, this.type, 'oa');
      this.colsV2Data = colVs;
      let type = this.type;
      console.log('colsV2Data', colVs);
      let fields = [];
      switch (type) {
        case 'update':
          fields = this.setFieldsDefaultVal(colVs._fieldInfo, this.activityData);
          break;
        case 'add':
          fields = colVs._fieldInfo;
          break;
        case 'detail':
          fields = this.setFieldsDefaultVal(colVs._fieldInfo, this.activityData);
          break;
        default:
          break;
      }
      console.log('fields:', fields, type);
      if (fields && Array.isArray(fields)) {
        fields = fields.filter((item, index) => {
          if (!item.value) {
            item.value = '';
          }
          if (item.column === 'activity_no') {
            item['disabled'] = true;
          }
          if (item['in_' + type] === 1) {
            return item;
          }
        });
        console.log('this.fields ', fields);
        this.fields = fields;
      }
    }
  },
  onLoad(option) {
    if (option.serviceName) {
      uni.setStorageSync('activeApp', 'oa');
      this.serviceName = option.serviceName;
      this.getBasicCfg();
      this.getColV2();
      // this.listConfig = JSON.parse(decodeURIComponent(decodeURIComponent(option.listConfig)));
    }
  }
};
</script>

<style lang="scss" scoped>
.form-wrap {
  height: calc(100vh - 100upx);
}
</style>
