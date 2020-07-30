<template>
  <view class="proc-wrap">
    <cu-custom bgColor="bg-blue" :isBack="true">
      <block slot="backText">返回</block>
      <block slot="content">流程详情</block>
    </cu-custom>
    <scroll-view scroll-x class="bg-white nav ">
      <view class="flex text-center">
        <view class="cu-item flex-sub" :class="index == TabCur ? 'text-orange cur' : ''" v-for="(item, index) in tabList" :key="index" @tap="tabSelect" :data-id="index">
          {{ item.label }}
        </view>
      </view>
    </scroll-view>
    <view class="detail-view content-box" v-if="TabCur === 0">
      <view class="detail-form"><bxform ref="bxDetailForm" :pageType="type" :BxformType="type" :fields="fields"></bxform></view>
    </view>
    <view class="steps-view content-box" v-if="TabCur === 1"><BxSteps :infoList="procBasicConfig.proCharData" :scroll="scroll" @clickSteps="clickSteps"></BxSteps></view>

    <view class="step-list  content-box" v-else-if="TabCur === 2">
      <view class="step-list-item" v-for="(item, index) in procRecord" :key="index" @click="toRecordDetail(item)">
        <view class="title">
          <text class="label">步骤名称：</text>
          <text class="value">{{ item.step_no_name }}</text>
        </view>
        <view class="status">
          <text class="label">处理结果：</text>
          <text class="value">{{ item.proc_result }}</text>
        </view>
        <view class="detail">
          <view class="detail-item">
            <view class="label" v-if="item.data_status">数据状态：{{ item.data_status }}</view>
            <view class="value"></view>
          </view>
          <view class="detail-item">
            <view class="label" v-if="item.create_user">审核人：{{ item.create_user }}</view>
            <view class="value"></view>
          </view>
          <view class="detail-item">
            <view class="label" v-if="item.create_user">创建人：{{ item.create_user }}</view>
            <view class="value"></view>
          </view>

          <view class="detail-item">
            <view class="label">审核时间：{{ item.modify_time.slice(0, 10) }}</view>
            <view class="value"></view>
          </view>
        </view>
      </view>
    </view>

    <view class="bottom-bar">
      <view class="" style="flex: 1;">
        <view class="">
          <text class="text-gray">当前步骤：</text>
          <text class="value text-blue">{{ currentStepInfo.step_name }}</text>
        </view>
        <view class="">
          <text class="text-gray">审批人：</text>
          <text>{{ currentStepInfo._approval_user }}</text>
        </view>
      </view>
      <view class="" v-if="procBasicConfig.proHanleData && procBasicConfig.proHanleData.authority">
        <text
          class="cu-btn bg-blue margin-right"
          v-if="procBasicConfig.proHanleData && procBasicConfig.proHanleData.activeStep !== 0"
          @click="showApprovalForm(currentStepInfo, 'approval')"
        >
          审批
        </text>
        <text
          class="cu-btn bg-blue margin-right"
          v-if="procBasicConfig.proHanleData && procBasicConfig.proHanleData.activeStep === 0"
          @click="showApprovalForm(currentStepInfo, 'approval')"
        >
          申请
        </text>
      </view>
    </view>
    <uni-popup ref="approvalPopup" type="bottom">
      <view class="form-view" v-if="stepsCfgData.length > 0">
        <view class="" style="width: 100%;" v-for="(item, index) in stepsCfgData" :key="index">
          <bxform :ref="'bxFormStep' + index" :pageType="item.formType" :BxformType="item.formType" :fields="item.fields"></bxform>
        </view>
        <bxform v-if="isHandler" ref="approvalForm" :pageType="'detail'" :BxformType="'detail'" :fields="approvalFormCfg"></bxform>
        <bxform
          v-if="procBasicConfig.proHanleData && procBasicConfig.proHanleData.activeStep !== 0 && !isHandler"
          ref="approvalForm"
          :pageType="'add'"
          :BxformType="'add'"
          :fields="approvalFormCfg"
        ></bxform>
        <view class="button-box" v-if="procBasicConfig.proHanleData && procBasicConfig.proHanleData.activeStep !== 0 && !isHandler">
          <text class="cu-btn bg-blue margin-right" @click="approvalForm">提交</text>
          <text class="cu-btn bg-green" @click="$refs.approvalPopup.close()">取消</text>
        </view>
        <text class="cu-btn bg-blue" @click="$refs.approvalPopup.close()" v-if="isHandler">确定</text>
      </view>
    </uni-popup>
    <uni-popup ref="recordPopup" type="bottom">
      <view class="form-view" v-if="currentRecord.fields && currentRecord.fields.length > 0">
        <bxform ref="bxFormRecord " pageType="detail" BxformType="detail" :fields="currentRecord.fields"></bxform>
        <view class="cu-btn bg-blue radius" @click="$refs.recordPopup.close()">确定</view>
      </view>
    </uni-popup>
  </view>
</template>

<script>
import bxform from '@/components/bx-form/bx-form.vue';
import uniPopup from '@/components/uni-popup/uni-popup.vue';
import BxSteps from '@/components/bx-steps/bx-steps.vue';
export default {
  components: { bxform, uniPopup, BxSteps },
  data() {
    return {
      TabCur: 1,
      scrollLeft: 0,
      tabList: [
        {
          label: '基本信息'
        },
        {
          label: '流程步骤'
        },
        {
          label: '流程审批记录'
        }
      ],
      isHandler: false,
      activityData: {},
      proc_instance_no: '',
      scroll: 0,
      type: 'add',
      procRecord: [],
      recordFields: [],
      currentRecord: {},
      procBasicConfig: {},
      colsV2Data: {},
      srvInfo: {
        app: '',
        serviceName: 'srvprocess_basic_cfg_select'
      },
      currentStepInfo: {}, //当前步骤的信息
      stepsCfgData: [], //当前步骤表单数据
      firstStepInfo: {}, //基础信息
      firstStepForm: [],
      formInfo: {
        serviceName: '',
        formType: ''
      },
      showProcApproval: false, //是否显示审批表单
      approvalVal: {
        proc_result: '',
        remark: '',
        turn_user_no: '',
        step_no: ''
      },
      fields: [],
      currentStepFields: [], //当前步骤的字段
      authority: false, //编辑权限
      approvalFormCfg: [
        {
          label: '意见',
          column: 'proc_result',
          value: '',
          type: 'radioFk',
          display: true,
          isRequire: true,
          isShowExp: [],
          options: [
            { value: 'pass', label: '已处理' },
            { value: 'turn', label: '转派' },
            { value: 'return_to_start', label: '退回开始' },
            { value: 'return_to_last', label: '退回' }
          ]
        },
        {
          label: '转派用户',
          column: 'turn_user_no',
          value: '',
          srvInfo: {
            serviceName: 'srvsso_user_select',
            appNo: 'sso',
            isTree: false,
            column: 'user_no',
            showCol: 'real_name' //要展示的字段
          },
          option_list_v2: {},
          type: 'cascader',
          display: true,
          isRequire: true,
          isShowExp: [{ colName: 'proc_result', ruleType: 'eq', value: 'turn' }],
          options: []
        },
        {
          label: '说明',
          column: 'remark',
          value: '',
          type: 'textarea',
          display: true,
          isRequire: false,
          isShowExp: [{ colName: 'proc_result', ruleType: 'eq', value: 'pass' }],
          options: []
        },
        {
          label: '说明',
          column: 'remark',
          value: '',
          type: 'textarea',
          display: true,
          isRequire: true,
          isShowExp: [{ colName: 'proc_result', ruleType: 'neq', value: 'pass' }],
          options: []
        }
      ]
    };
  },
  methods: {
    tabSelect(e) {
      this.TabCur = Number(e.currentTarget.dataset.id);
      this.scrollLeft = (e.currentTarget.dataset.id - 1) * 60;
    },
    toRecordDetail(data) {
      //查看流程审批记录详情
      console.log(data);
      this.currentRecord = data;
      this.$refs.recordPopup.open();
    },
    hideApprovalForm() {
      this.$refs.approvalPopup.close();
    },
    showApprovalForm(data, type) {
      this.stepsCfgData = [];
      if (data.state === '已处理') {
        this.isHandler = true;
      } else {
        this.isHandler = false;
      }
      this.getApprovalForm(data).then(e => {
        console.log('eeeeeeeeeeeeee', e, e.length, e[0].fields);
        this.stepsCfgData = [];
        if (type && type == 'approval') {
          this.approvalFormCfg.forEach(item => {
            item.value = '';
          });
          if (e.length > 0 && e[0].fields) {
            e.forEach(item => {
              item.fields.forEach(item2 => {
                item2.value = '';
              });
            });
          }
        }
        // if (e.length > 0 && e[0].fields) {
        this.stepsCfgData = e;
        this.$refs.approvalPopup.open();
        // } else if (e.length > 0 && !e[0].fields) {
        // }
      });
    },
    clickSteps(e) {
      let approvalFormCfg = this.approvalFormCfg;
      let stepData = e.data;
      let recordData = this.procRecord;
      recordData.forEach(record => {
        if (stepData.step_no === record.step_no) {
          // stepData = Object.assign(record,stepData)
          stepData.remark = record.remark;
          stepData.proc_result = record.proc_result;
          console.log('step,record', stepData);
          Object.keys(stepData).forEach(key => {
            this.approvalFormCfg.forEach((field, index) => {
              if (field.column === key) {
                field['value'] = stepData[key];
                this.$set(this.approvalFormCfg, index, field);
              }
            });
          });
        }
      });
      if (e.index !== 0 && e.index <= this.scroll) {
        if (e.data.state !== '已处理') {
          this.showApprovalForm(e.data, 'approval');
        } else {
          this.showApprovalForm(e.data);
        }
      } else if (e.index === 0) {
        this.TabCur = 0;
      }
      // approvalFormCfg
      console.log('clickSteps', e);
    },
    async getBasicCfg(proc_instance_no) {
      // srvprocess_basic_cfg_select 流程初始化数据查询
      let serviceName = this.srvInfo.serviceName;
      uni.showLoading({});
      let req = { serviceName: 'srvprocess_basic_cfg_select', colNames: ['*'], condition: [{ colName: 'proc_instance_no', ruleType: 'eq', value: proc_instance_no }] };
      let res = await this.onRequest('select', 'srvprocess_basic_cfg_select', req, 'oa');
      uni.hideLoading();
      if (res.data.state === 'SUCCESS') {
        this.procBasicConfig = res.data;
        this.activityData = res.data.mainData;
        // this.getCurStepConfig(res.data['proCharData'][res.data['proHanleData']['activeStep']]);
        this.getApprovalForm(res.data['proCharData'][res.data['proHanleData']['activeStep']]); //获取当前步骤的信息
        this.getCurStepConfig(res.data['proCharData'][0], 'firstStep'); //获取第一步信息
        this.currentStepInfo = res.data['proCharData'][res.data['proHanleData']['activeStep']];
        this.firstStepInfo = res.data['proCharData'][0];
        this.scroll = res.data.proHanleData.activeStep;
      }
    },
    async getApprovalForm(e) {
      console.log('getApprovalForm', e);
      let cfg = e.biz_cfg_data; //表单配置
      let cfgData = [
        {
          type: '',
          serviceName: '',
          fields: [],
          formData: {}
        }
      ];
      if (cfg && cfg.length > 0) {
        uni.showLoading({
          title: ''
        });
        for (let i = 0; i < cfg.length; i++) {
          const item = cfg[i];
          if (item.type === 'form') {
            if (item._type_form && e.state !== '已处理') {
              let serviceName = item[`${item._type_form}_service`];
              let type = item._type_form;
              let fields = await this.getColV2(serviceName, item._type_form);
              cfg[i]['fields'] = fields;
              cfg[i]['formType'] = type;
              console.log('fields111111', fields);
            } else if (item.select_service && e.state === '已处理' && item._type_form) {
              let serviceName = item.select_service;
              let fields = await this.getColV2(serviceName, 'detail');
              // cfg[i]['fields'] = fields;
              cfg[i]['formType'] = 'detail';
              console.log('fields22222', fields);
              let data = await this.getDetail(serviceName, fields);
              // this.getDetail(serviceName, fields).then(data => {
              console.log('getDetail(serviceName)', data);
              Object.keys(data).forEach(key => {
                fields.forEach((field, index) => {
                  if (field.column === key) {
                    field['value'] = data[key];
                    field['disabled'] = true;
                    this.$set(fields, index, field);
                    cfg[i]['fields'] = fields;
                  }
                });
              });
              // });
            }
          }
        }
        uni.hideLoading();
        // this.stepsCfgData = cfg;
        return cfg;
      }
    },
    async getProcRecord() {
      //查找流程审批记录
      uni.showLoading({});
      let req = {
        serviceName: 'srvprocess_record_select',
        colNames: ['*'],
        condition: [{ colName: 'proc_instance_no', value: this.proc_instance_no, ruleType: 'eq' }],
        order: [{ colName: 'id', orderType: 'desc' }]
      };
      let res = await this.onRequest('select', 'srvprocess_record_select', req, 'oa');
      uni.hideLoading();
      if (res.data.state === 'SUCCESS') {
        this.procRecord = res.data.data;
        this.getColV2(req.serviceName, 'detail').then(cols => {
          console.log('recordFields', cols);
          this.recordFields = cols;
          this.procRecord.forEach((item, i) => {
            let recordFields = this.deepClone(cols);
            if (cols && Array.isArray(cols)) {
              Object.keys(item).forEach(key => {
                recordFields.forEach((field, index) => {
                  if (field.column === key) {
                    field['value'] = item[key];
                  }
                });
              });
              item['fields'] = recordFields;
              this.$set(this.procRecord, i, item);
            }
          });
        });
      }
    },
    async getDetail(serviceName, fields) {
      uni.showLoading({});
      let col = fields.map(item => item.column);
      let req = {
        serviceName: serviceName,
        condition: [{ colName: 'proc_instance_no', ruleType: 'eq', value: this.proc_instance_no }],
        colNames: col,
        hisVer: true
      };
      let res = await this.onRequest('select', req.serviceName, req, 'oa');
      uni.hideLoading();
      if (res.data.state === 'SUCCESS') {
        console.log('getDetail111', res.data.data);
        if (res.data.data.length > 0) {
          return res.data.data[0];
        }
      }
    },
    getCurStepConfig(e, type) {
      if (e && e.state !== '未开始') {
        console.log('getCurStepConfig', e);
        if (e.biz_cfg_data && e.biz_cfg_data.length > 0) {
          const biz_cfg = e.biz_cfg_data;
          biz_cfg.map(item => {
            if (item._type_form) {
              this.formInfo.formType = item._type_form;
              this.type = item._type_form;
              this.formInfo.serviceName = item[`${item._type_form}_service`];
              this.getColV2(this.formInfo.serviceName, item._type_form).then(fields => {
                if (type === 'firstStep') {
                  this.fields = fields;
                  Object.keys(this.procBasicConfig.mainData).forEach(key => {
                    this.fields.forEach((field, index) => {
                      if (field.column === key) {
                        field['value'] = this.procBasicConfig.mainData[key];
                        this.$set(this.fields, index, field);
                      }
                    });
                  });
                } else {
                  this.currentStepFields = fields;
                }
              });
            } else if (item.select_service) {
              this.formInfo.formType = 'detail';
              this.type = 'detail';
              this.formInfo.serviceName = item.select_service;
              this.getColV2(this.formInfo.serviceName, this.type).then(fields => {
                if (type === 'firstStep') {
                  this.fields = fields;
                  Object.keys(this.procBasicConfig.mainData).forEach(key => {
                    this.fields.forEach((field, index) => {
                      if (field.column === key) {
                        field['value'] = this.procBasicConfig.mainData[key];
                        this.$set(this.fields, index, field);
                      }
                    });
                  });
                  // this.getDetail(item, 'firstStep');
                  // this.TabCur = 0
                } else {
                  this.currentStepFields = fields;
                }
              });
            }
          });
        }
      }
    },
    approvalForm() {
      // 提交审批
      let self = this;
      if (this.procBasicConfig.proHanleData.activeStep === 0) {
        //重新申请
        uni.showLoading({});
        for (let i = 0; i < this.stepsCfgData.length; i++) {
          let ref = 'bxFormStep' + i;
          let item = this.stepsCfgData[i];
          if (item.formType) {
            let serviceName = item[`${item.formType}_service`];
            let itemData = self.$refs[ref][0].getFieldModel();
            if (!itemData) {
              itemData = this.activityData;
            }
            let req = [
              {
                serviceName: 'srvoa_issue_info_update',
                condition: [{ colName: 'proc_instance_no', ruleType: 'eq', value: this.proc_instance_no }],
                proc_instance_no: this.proc_instance_no,
                data: [itemData]
              }
            ];
            this.onRequest('apply', serviceName, req, 'oa').then(res => {
              uni.hideLoading();
              if (res.data.state === 'SUCCESS') {
                console.log(res.data, 'res.data');
                uni.showToast({
                  title: res.data.resultMessage,
                  icon: 'none'
                });

                uni.showModal({
                  title: '提示',
                  content: res.data.resultMessage,
                  showCancel: false,
                  success(res) {
                    if (res.confirm) {
                      self.hideApprovalForm();
                      self.getBasicCfg(self.proc_instance_no);
                      self.getProcRecord(self.proc_instance_no);
                    }
                  }
                });
              }
            });
          }
        }
      } else {
        let approval = self.$refs.approvalForm.getFieldModel();
        console.log('approval', approval);
        let child_data_list = [];
        let id = this.procBasicConfig.mainData.id;
        let stepsCfgData = this.stepsCfgData;
        if (approval.proc_result === 'pass') {
          for (let i = 0; i < this.stepsCfgData.length; i++) {
            let ref = 'bxFormStep' + i;
            let item = this.stepsCfgData[i];
            if (item.formType) {
              let serviceName = item[`${item.formType}_service`];
              let obj = {
                serviceName: item[`${item.formType}_service`],
                data: [self.$refs[ref][0].getFieldModel()],
                condition: [
                  {
                    colName: 'id',
                    ruleType: 'in',
                    value: self.procBasicConfig.mainData.id
                  }
                ]
              };
              child_data_list.push(obj);
            }
          }
        }
        // if (this.procBasicConfig.proHanleData.activeStep === 1) {
        //   child_data_list = [];
        // }
        console.log(child_data_list, 'child_data_list');
        let reqData = [
          {
            data: [
              {
                child_data_list: child_data_list,
                key: approval.proc_result,
                proc_result: approval.proc_result,
                remark: approval.remark,
                turn_user_no: approval.turn_user_no
              }
            ],
            proc_instance_no: self.proc_instance_no,
            step_no: self.currentStepInfo.step_no
          }
        ];
        uni.showLoading({});
        let url = this.getServiceUrl('oa', 'approval', 'process');
        self.onRequest('process', 'approval', reqData, 'oa').then(res => {
          uni.hideLoading();
          if (res.data.state === 'SUCCESS') {
            console.log(res.data);
            uni.showToast({
              title: res.data.resultMessage,
              icon: 'none'
            });
          }
          this.hideApprovalForm();
          this.getBasicCfg(this.proc_instance_no);
          this.getProcRecord(this.proc_instance_no);
        });
        console.log(`reqData`, reqData);
      }
    },
    async getColV2(serviceName, type) {
      let colVs = await this.getServiceV2(serviceName, type, type, 'oa');
      this.colsV2Data = colVs;
      // let type = this.type;
      console.log('colsV2Data', colVs);
      let fields = [];
      switch (type) {
        // case 'update':
        //   fields = this.setFieldsDefaultVal(colVs._fieldInfo, this.activityData);
        //   break;
        case 'update':
          fields = colVs._fieldInfo;
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
      if (this.scroll === 0 && type == 'update') {
        fields = this.setFieldsDefaultVal(colVs._fieldInfo, this.activityData);
      }
      if (fields && Array.isArray(fields)) {
        fields = fields.filter((item, index) => {
          if (!item.value) {
            item.value = '';
          }
          // if (item.type === 'Note') {
          //   item.type = 'textarea';
          // }
          // if (item['in_' + type] === 1) {
          return item;
          // }
        });

        // this.fields = fields;
      }
      console.log('colsV2Datafields', fields);
      return fields;
    }
  },
  onLoad(option) {
    if (option.proc_instance_no) {
      this.proc_instance_no = option.proc_instance_no;
      this.srvInfo.app = uni.getStorageSync('activeApp');
      this.getBasicCfg(option.proc_instance_no);
      this.getProcRecord(option.proc_instance_no);
    }
  }
};
</script>

<style scoped lang="scss">
.proc-wrap {
  padding-bottom: 150upx;
  position: relative;
  .scroll-fixed {
    top: 90upx;
    z-index: 1024;
    position: fixed;
  }
  .uni-popup {
    z-index: 999;
  }
  /deep/ .uni-scroll-view {
    height: auto;
  }
}
.steps-view {
  // margin-top: 100upx;
  // padding-bottom: 150upx;
}
.flow-view {
  width: 100%;
  // min-height: 80vh;
  margin-top: 10upx;
  background-color: #fff;
  overflow: hidden;
  .scroll-view {
    padding: 30upx 10upx;
  }
  .cu-timeline {
    .cu-item {
      padding: 5px 15px 5px 54px;
      position: relative;
      display: block;
      z-index: 0;
    }
    .head {
      display: flex;
    }
    .name {
      flex: 1;
      font-size: 34upx;
      line-height: 60upx;
      font-weight: 700;
    }
    .state {
      flex: 1;
      font-size: 30upx;
      line-height: 60upx;
    }
  }
}
.step-list {
  width: 100%;
  display: flex;
  flex-direction: column;
  .step-list-item {
    background-color: #fff;
    width: calc(100% - 20upx);
    display: flex;
    flex-direction: column;
    margin: 10upx;
    padding: 20upx;
    border-radius: 10upx;
    .title {
      font-size: 32upx;
      line-height: 50upx;
      font-weight: bold;
    }
    .detail {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-between;
      .detail-item {
        min-width: 40%;
      }
    }
  }
}
.detail-view {
  margin-top: 10upx;
  padding-bottom: 150upx;
  min-height: 100vh;
  background-color: #fff;
  display: flex;
  flex-direction: column;
}
.cu-dialog {
  height: auto;
}
.form-view {
  background-color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  max-height: calc(100vh - 100px);
  overflow-y: scroll;
  padding: 30upx 0 100upx;
  .button-box {
    height: 100upx;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 20upx 0 40upx;
  }
  .cu-btn {
    max-width: 60%;
    min-height: 60upx;
    margin-top: 50upx;
  }
}
.bottom-bar {
  background-color: #fff;
  z-index: 20;
  width: 100%;
  height: 100upx;
  padding-left: 20upx;
  display: flex;
  // flex-direction: column;
  // justify-content: center;
  align-items: center;
  border-top: 1px solid rgba($color: #999, $alpha: 0.5);
  position: fixed;
  bottom: 0;
  .text-blue {
    font-weight: bold;
    font-size: 32upx;
  }
}
.content-box {
  margin-bottom: 100upx;
}
</style>
