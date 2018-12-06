<template>
  <div class="page">
    <div class="data-title">
      <h2 class="page-title">爬虫服务监控</h2>
      <div class="search">
        <div class="s-left">
          <div class="s-dot">
            <span>修改时间: </span>
            <el-date-picker
              v-model="listQuery.startTime"
              type="date"
              placeholder= "开始时间"
              @change="changeStartTime"
              :picker-options="pickerOptions0">
            </el-date-picker>
            -
            <el-date-picker
              v-model="listQuery.endTime"
              type="date"
              placeholder="结束时间"
              @change="changeEndTime"
              :picker-options="pickerOptions1">
            </el-date-picker>
          </div>
          <div class="s-dot">
            <span></span>
            <button class="s-btn" @click="search">搜索</button>
          </div>
        </div>
        
      </div>
    </div>
    
  </div>
</template>

<script>
  import {
    getCrawlerService,
    delCrawler
  } from 'api/dataMonitor'

  export default {
    data() {
      return {
//        arr: ['装船', '重箱进场', '申报', 'VGM回执','重箱进场', '申报', 'VGM回执','VGM回执','重箱进场', '申报', 'VGM回执'],
        tableData: [],
        pickerOptions0: {
          disabledDate: (time) => {
            if (this.listQuery.endTime != "") {
              return time.getTime() > Date.now() || time.getTime() > this.listQuery.endTime;
            } else {
              return time.getTime() > Date.now();
            }
          },
        }, //超时当前时间后，禁用时间
        pickerOptions1: {
          disabledDate: (time) => {
            return time.getTime() < this.listQuery.startTime || time.getTime() > Date.now();
          }
        },
        listQuery: {
          page: 1,
          limit: 5,
          startTime: '',
          endTime: '',
          searchKey: ''
        },
        total: null,
        sels: [],//选中的值显示
        disabled:true,
        sort: 'createAt',
        order: 'descending',
      }
    },
   
    methods: {
     
      
        //计算相隔天数
      timeFn(dateBegin, dateEnd){

//        let dateBegin = new Date(dateBegin);//获取当前时间
//        let dateEnd = new Date(dateEnd);
        let dateDiff = new Date(dateEnd).getTime() - new Date(dateBegin).getTime();//时间差的毫秒数
        let dayDiff = Math.floor(dateDiff / (24 * 3600 * 1000));//计算出相差天数
//        let leave1=dateDiff%(24*3600*1000)    //计算天数后剩余的毫秒数
//        let hours=Math.floor(leave1/(3600*1000))//计算出小时数
//        //计算相差分钟数
//        let leave2=leave1%(3600*1000)    //计算小时数后剩余的毫秒数
//        let minutes=Math.floor(leave2/(60*1000))//计算相差分钟数
//        //计算相差秒数
//        let leave3=leave2%(60*1000)      //计算分钟数后剩余的毫秒数
//        let seconds=Math.round(leave3/1000)
        if(dayDiff < 0){
          return '0'
        }else if(dayDiff > 31){
          return '1';
        }
      },

      //	选择开始时间
      changeStartTime(options) {
        let _this =this;
        if(options) {
          _this.listQuery.startTime = options;
        } else {
          _this.listQuery.startTime = ''
        }
        /*if(new Date(_this.listQuery.startTime).getTime() > new Date(_this.listQuery.endTime).getTime()) {
        return '0';
      } else if(months >= 12) {
        return '1';
      } else {}*/
//        console.log(_this.listQuery.startTime , _this.listQuery.endTime, "时间查询---" );
//        let isshow = CF.dateMethod(_this.listQuery.startTime, _this.listQuery.endTime);
        /*if(isshow == 0) {
          _this.$alert('开始时间不能晚于结束时间', '提示', {
            confirmButtonText: '确定',
          });
          _this.listQuery.startTime = '';
        } else {}*/
      },
      //选择结束时间
      changeEndTime(options) {
        let _this =this;
        if(options) {
          _this.listQuery.endTime = options;
        } else {
          _this.listQuery.endTime = '';
        }
      },
     
      
      // 通知 —— 悬浮出现在页面右上角，显示全局的通知提醒消息。
      unDatanotification() {
        this.$notify.info({
          title: '消息',
          message: '您好，暂无数据！'
        });
      },
    
      search() {
       
        let isTime = this.timeFn(this.listQuery.startTime, this.listQuery.endTime)
        if(isTime == 0){
          this.$alert('开始时间不能晚于结束时间', '提示', {
            confirmButtonText: '确定',
          });
          this.listQuery.endTime = '';
          return false;
        }else if(isTime == 1){
          this.$alert('时间范围不能超过31天', '提示', {
            confirmButtonText: '确定',
          })
          this.listQuery.endTime = '';
          return false;
        }else{}
        if(!this.listQuery.startTime && !this.listQuery.endTime){
          this.$alert('修改时间不能为空', '提示', {
            confirmButtonText: '确定',
          });
          return false;
        }
        if(!this.listQuery.endTime && this.listQuery.startTime){
          this.listQuery.endTime = this.listQuery.startTime
        }
        if(!this.listQuery.startTime && this.listQuery.endTime){
          this.listQuery.startTime = this.listQuery.endTime
        }
        this.getCrawlerService();
        setTimeout(() => {
          if(this.tableData.length==0 || this.tableData == 'undefined'){
            this.unDatanotification()
          }else{}
        },1000) 
      },
    }
  };
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .pagination{
    margin-top: -40px;
  }
  .el-input{
    width: 200px !important;
  }
  .data-title{
    height: 190px;
    background: #fff;
    h2{
      height: 65px;
      line-height: 65px;
      font-weight: 400;
    }
  }
  .search{
    padding-top: 15px;
    padding-left: 20px;
  }
  .s-left, .s-right{
    float: left;
    width: 55%;
  }
  .s-right{
    width: 45%;
  }
  .s-dot{
    padding-bottom: 22px;
    span{
      display: inline-block;
      width: 70px;
      text-align: right;
    }
    span.big{
      width: 120px;
    }
  }
  .page-content{
   
    border: 1px solid #D9D9D9;
    padding: 0;
  }
  .data-title2{
    width: 100%;
    height: 40px;
    line-height: 40px;
    background: #FBFBFB;
    border-bottom: 1px solid #D9D9D9;
    font-size: 16px;
    padding-left: 20px;
    /*float: left;*/
  }
  .form-btn{
    float: right;
    text-align: right;
    height: 80%;
    line-height: 1.5em;
    margin: 5px;
    box-sizing: border-box;
  }
  .data-right{
    margin-left: 20px;
    margin-top: -20px;
  }
  .data-content{
    display: flex;
    display: -webkit-flex;
    height: 286px;
    width: 100%;
  }
  .data-left{
    width: 376px;
    position: relative;
  }
  .data-right{
    flex: 1;
    -webkit-flex: 1;
    padding-top: 29px;
    padding-right: 20px;
  }
  #stateEC1_1, #stateEC1_2{
     width: 100%;
     height: 100%;
  }
  .echarts-tooltip{
    position: absolute;
    left: 50%;
    bottom: 10px;
    font-size: 10px;
    margin-left: -78px;
    span:nth-of-type(1), span:nth-of-type(3){
      display: inline-block;
      width: 4px;
      height: 4px;
      background: #314552;
      vertical-align: middle;
      margin-bottom: 2px;
    }
    span:nth-of-type(3){
      margin-left: 10px;
      background: #C8843A;
      
    }
  }
</style>
