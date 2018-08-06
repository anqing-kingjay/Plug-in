<template>
  <div class="record" :class="{apprecord:!isWeixin}">
    <van-nav-bar title="交费记录" left-arrow v-if="!isWeixin" @click-left="onBack">
      <img src="../assets/images/arrow.png" alt="" slot="left" style="width:18px;vertical-align: middle;">
    </van-nav-bar>
    <div v-if="isServer==='has'">
      <div class="record_title" :class="{apptitle:!isWeixin}">
        <van-cell-group>
          <van-cell title="服务单位" :is-link="serverInfo.isServerList" :value="serverInfo.serverName" @click="onPopupShow" />
        </van-cell-group>
      </div>
      

      <div v-if="isRecord" class="record_list_box">
          <quickloadmore
            ref="vueLoad" 
            :bottomMethod="handleBottom" 
            :bottomStatusChange="handleBottomStatusChange" 
            :disableTop="true" 
            :disableBottom="false">
            
            <div class="record_list_wrap" v-for="(item,index) in recordList" :key="index" @click="onDetails(item.payNo)">
              <div class="list_header">
                <div class="number">订单编号：{{item.payNo}}</div>
                <div class="list_status" v-if="item.payStatus == 1">已支付</div>
                <div class="list_status" v-else>未支付</div>
              </div>
              <div class="list_cont" v-for="(citem,cindex) in item.detailList" :key="cindex">
                <div class="info">
                  <p class="title">{{citem.productName}} {{citem.addrName}}</p>
                  <p class="time">服务期限：{{citem.startDate}}至{{citem.endDate}}</p>
                </div>
                <div class="year_wrap"> <span class="year">x{{citem.year}}年</span></div>
              </div>
              <div class="list_footer">
                <span v-if="item.payStatus == 1">合计：<b>&#65509;{{item.payTotal}}</b></span>
                <span v-else>未支付</span>
              </div>
            </div>
          </quickloadmore>

      </div>

      <div v-else>
        <img src="../assets/images/noproduct.png" alt="" class="no_product">
        <p class="no_product_text">该服务单位下暂无交费记录</p>
      </div>
    </div>
    <div v-else-if="isServer==='no'">
        <img src="../assets/images/noproduct.png" alt="" class="no_product">
        <p class="no_product_text">暂无交费记录</p>
    </div>
    <div v-else></div>

    

    <van-popup v-model="popupShow" position="bottom">
      <van-picker 
      show-toolbar 
      @confirm="onPickerConfirm"
      @cancel="popupShow = false"
      :item-height="Number(46)" 
      :visible-item-count="Number(4)"
      :columns="serverInfo.columns" />
    </van-popup>
  </div>
</template>

<script>
import { Cell, CellGroup,Popup,Picker,NavBar} from 'vant';
import {getRecordList,getServerList} from '../server';
import quickloadmore from './load.vue';

export default {
  name:"record",
  components: {
    [Cell.name]: Cell ,
    [CellGroup.name]: CellGroup,
    [Popup.name]: Popup ,
    [Picker.name]: Picker,
    [NavBar.name]: NavBar,
    quickloadmore
  },
  data(){
    return {
      isWeixin:this.isWeixin,
      current:2,
      companyName:"",
      popupShow:false,
      recordList:[],
      // recordList:[
      //   {
      //     "detailList": [{//订单记录明细列表
      //       "addrName": "开票机0",//开票机号的地址说明
      //       "endDate": "2015-08-03",//服务结束日期
      //       "price": "400",//产品单价
      //       "productName": "防伪税控开票系统",//产品名称
      //       "startDate": "2014-08-03",//服务开始日期
      //       "year": "1"//交费年数
      //     }],
      //     "payNo": "jf201408041502208122222222222222222",//交费编号
      //     "payStatus": "1",//支付状态：0未支付 1已支付
      //     "payTotal": "580",//支付金额
      //   },
      //   {
      //     "detailList": [{//订单记录明细列表
      //       "addrName": "开票机0",//开票机号的地址说明
      //       "endDate": "2015-08-03",//服务结束日期
      //       "price": "400",//产品单价
      //       "productName": "防伪税控开票系统",//产品名称
      //       "startDate": "2014-08-03",//服务开始日期
      //       "year": "1"//交费年数
      //     }],
      //     "payNo": "jf201408041502208122222222222222222",//交费编号
      //     "payStatus": "1",//支付状态：0未支付 1已支付
      //     "payTotal": "580",//支付金额
      //   },
      //   {
      //     "detailList": [{//订单记录明细列表
      //       "addrName": "开票机0",//开票机号的地址说明
      //       "endDate": "2015-08-03",//服务结束日期
      //       "price": "400",//产品单价
      //       "productName": "防伪税控开票系统",//产品名称
      //       "startDate": "2014-08-03",//服务开始日期
      //       "year": "1"//交费年数
      //     }],
      //     "payNo": "jf201408041502208122222222222222222",//交费编号
      //     "payStatus": "1",//支付状态：0未支付 1已支付
      //     "payTotal": "580",//支付金额
      //   },
      //   {
      //     "detailList": [{//订单记录明细列表
      //       "addrName": "开票机0",//开票机号的地址说明
      //       "endDate": "2015-08-03",//服务结束日期
      //       "price": "400",//产品单价
      //       "productName": "防伪税控开票系统",//产品名称
      //       "startDate": "2014-08-03",//服务开始日期
      //       "year": "1"//交费年数
      //     }],
      //     "payNo": "jf201408041502208122222222222222222",//交费编号
      //     "payStatus": "1",//支付状态：0未支付 1已支付
      //     "payTotal": "580",//支付金额
      //   },
      //   {
      //     "detailList": [{//订单记录明细列表
      //       "addrName": "开票机0",//开票机号的地址说明
      //       "endDate": "2015-08-03",//服务结束日期
      //       "price": "400",//产品单价
      //       "productName": "防伪税控开票系统",//产品名称
      //       "startDate": "2014-08-03",//服务开始日期
      //       "year": "1"//交费年数
      //     }],
      //     "payNo": "jf201408041502208122222222222222222",//交费编号
      //     "payStatus": "1",//支付状态：0未支付 1已支付
      //     "payTotal": "580",//支付金额
      //   },
      // ],
      isServer:"",
      isRecord:true,
      serverInfo:{
        columns:[],//服务单位名称数组
        serverArry:[],//服务单位id数组
        companyArry:[],//公司id数组
        serverGroupArry:[],//服务单位组织id数组
        serverName:"",//选中的服务单位名称
        comServiceId:"",//选中的服务单位id
        isServerList:false, //判断是否有多个服务单位,
        companyId:"",//公司id
        serverGroupId:"",//服务单位组织id
      },
    }
  },
 
  created() {
    document.title = "交费记录"
    //获取缴费记录列表

    if(this.$route.query.comServiceId){
      this.isServer = 'has';
    }else{
      this.isServer = 'no';
      // this.isServer = 'has';
      return
    }
    
    //针对从订单详情返回记录的状态
    if(localStorage.recordServerName){
      this.serverInfo.serverName = localStorage.recordServerName;
      this.serverInfo.comServiceId = localStorage.recordComServiceId;
    }else{
      this.serverInfo.serverName = this.$route.query.serverName;
      this.serverInfo.comServiceId = this.$route.query.comServiceId;
    }

    getRecordList({params:{comServiceId:this.serverInfo.comServiceId}}).then(res =>{
      let datas = res.data
      if(datas.status ==1 ){
        if(datas.data && datas.data.length){
          this.isRecord = true;
          this.canLoading = true;
          this.recordList = datas.data
        }else{
          this.isRecord = false;
        }
      }else{
        this.Toast(datas.msg)
      }
    });

    this.getProduct()
  },
  methods:{
     handleBottomStatusChange(status) {
      // status监控上拉加载状态--等待/加载/没有更多数据  
      const BOTTOMSTATUS = {
        wait: "wait",
        loading: "loading",
        nodata: "nodata"
      };
      console.log(status, "bottomchange");
    },
    handleBottom() {
       console.log(11)
       
      // 上拉加载
      getRecordList({params:{comServiceId:this.serverInfo.comServiceId,current:this.current}}).then(res =>{
        let datas = res.data
        if(datas.status ==1 ){
          if(datas.data && datas.data.length){
            this.current++;
            console.log(this.current)
            this.recordList = this.recordList.concat(datas.data)
            this.$refs.vueLoad.onBottomLoaded();
          }else{
            this.$refs.vueLoad.onBottomLoaded(false);
          }
        }else{
          this.$refs.vueLoad.onBottomLoaded(false);
        }
      });
      // setTimeout(() =>{
      //   this.recordList.push(
      //   {
      //     "detailList": [{//订单记录明细列表
      //       "addrName": "开票机0",//开票机号的地址说明
      //       "endDate": "2015-08-03",//服务结束日期
      //       "price": "400",//产品单价
      //       "productName": "防伪税控开票系统",//产品名称
      //       "startDate": "2014-08-03",//服务开始日期
      //       "year": "1"//交费年数
      //     }],
      //     "payNo": "jf20140804150220812",//交费编号
      //     "payStatus": "1",//支付状态：0未支付 1已支付
      //     "payTotal": "580",//支付金额
      //   }
      // )
      // this.$refs.vueLoad.onBottomLoaded();
      // },200)
      
    },
    
    getProduct(){//获取服务单位信息及产品信息
      getServerList({params:{'userId':this.$route.query.uid}}).then(res=>{
        let datas = res.data;
        if(datas.status == 1){
          let list = datas.data;
          if(list && list.length){
            list.forEach(element => {
              this.serverInfo.columns.push(element.companyname);
              this.serverInfo.serverArry.push(element.id);
            });
            this.serverInfo.isServerList = this.serverInfo.columns.length>1?true:false;
          }
        }else{
          this.Toast(datas.msg)
        }
      })
    },
    onPopupShow(){
      if(this.serverInfo.isServerList){
        this.popupShow = true
      }
    },
    onPickerConfirm(value, index){
      this.popupShow = false;
      this.companyName = value
    },
    onPickerConfirm(value, index){//服务单位弹出层选择确认事件
      this.popupShow = false;
      this.serverInfo.serverName = value;
      this.serverInfo.comServiceId = this.serverInfo.serverArry[index];

      getRecordList({params:{comServiceId:this.serverInfo.comServiceId}}).then(res =>{
        let datas = res.data
        if(datas.status ==1 ){
          if(datas.data && datas.data.length){
            this.isRecord = true;
            this.recordList = datas.data
          }else{
            this.isRecord = false;
          }
        }else{
          this.Toast(datas.msg)
        }
      });

    },
    onDetails(num){
      localStorage.recordServerName = this.serverInfo.serverName;
      localStorage.recordComServiceId = this.serverInfo.comServiceId;
      this.$router.push({path:"/orderDetails",query:{payNo:num,comServiceId:this.serverInfo.comServiceId,hasParent:'true'}})
    },
    onBack(){
      history.back();
    },
  }
};
</script>

<style lang='less' scoped>
.record{
  height: 100%;
  padding-top: 44px;
  box-sizing: border-box;
  position: relative;

  .record_title{
    position: fixed;
    width: 100%;
    top: 0;
    z-index: 100;
  }
  .apptitle{
    top: 46px;
  }
  .record_list_box{
    height:600px;
    height:100vh;
    overflow: hidden;
  }
  .record_list_wrap{
    margin-bottom: .1rem;
    background: white;
    .list_header{
      display: flex;
      padding: 0 .15rem;
      line-height: .36rem;
      border-bottom: 1px solid #F6F6F6;
      .number{
        flex: 4;
      }
      .list_status{
        flex: 1;
        text-align: right
      }
    }
    .list_cont{
      display: flex;
      padding: .12rem .15rem;
      border-bottom: 1px solid #F6F6F6;
      .info{
        flex: 4;
        .time{
          color: #9B9B9B;
          font-size: 12px;
          margin-top: .08rem;
        }
      }
      .year_wrap{
        flex: 1;
        color: #9B9B9B;
        font-size: 12px;
        position: relative;
        .year{
          position: absolute;
          right: 0;
          bottom: 0;
        }
      }
    }
    .list_footer{
      padding: 0 .15rem;
      height: .36rem;
      line-height: .36rem;
      text-align: right;
      b{
        color: #FE8604
      }
    }
  }
  .no_product{
    display: block;
    width: .97rem;
    margin: .81rem auto .2rem;
  }
  .no_product_text{
    color:rgba(155,155,155,1);
    text-align: center
  }
}
.apprecord{
  padding-top: 90px
}
</style>
