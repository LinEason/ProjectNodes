<template>

  <div class="personal-container">

    <div class="common-wrapper">

      <div class="vreset-header">
        <inside-header></inside-header>
      </div>

      <div class="vpage-scroll">

        <div class="invoice-container">

          <router-link to="/invoice_history" tag="div" class="invoice-access">
            <div class="vweui-flex">
              <div class="weui-cell__bd"><p>开票历史</p></div>
              <div class="weui-cell__ft">
                <span class="next-arrow-module">
                  <i class="iconfont icon-back"></i>
                </span>
              </div>
            </div>
          </router-link>

          <div class="invoice-content">

            <div class="invoice-module" v-for="(item,index) in orders_invoice" :key="index">
              <div class="vorder-panel">
                <div class="check-model">
                  <label class="vweui-flex weui-check__label" :for="item.orderNo">
                    <div class="weui-cell__hd">
                        <input 
                          type="checkbox" 
                          class="weui-check" 
                          name="orders" 
                          :id="item.orderNo"
                          :value="item.orderNo"
                          v-model="checked_order"
                          @change="checkedOrder"
                        />
                        <i class="weui-icon-checked"></i>
                    </div>
                    <div class="weui-cell__bd">
                        <p>订单编号 {{item.orderNo}}</p>
                    </div>
                  </label>
                </div>

              </div>
              <order-info :product_info="item.orderItemEntitys"></order-info>
              <div class="vorder-base">
                <div class="base-pay"><p>应付金额：<span class="price">{{item.payMoney | currency('¥')}}</span></p></div>
              </div>
            </div>

          </div>

        </div>

        <p class="no-data" v-show="isnodata">暂无数据</p>

        <div v-show="loading" class="weui-loadmore">
          <i class="weui-loading"></i>
          <span class="weui-loadmore__tips">正在加载</span>
        </div>
        <!-- scroll -->
        <div
          class="m-infinite"
          v-show="!isnodata && !loading"
          v-infinite-scroll="loadMore"
          infinite-scroll-disabled="scrollDisabled"
          infinite-scroll-distance="distance"
          infinite-scroll-immediate-check = "false"
        >
          <div v-show="infiniteMsgShow" class="weui-loadmore">
            <i class="weui-loading"></i>
            <span class="weui-loadmore__tips">正在加载</span>
          </div>
          <p v-show="!infiniteMsgShow" class="tips">———————— 没有更多数据 ————————</p>
        </div>


      </div>

      <div class="invoice-navbar">
        <div class="vweui-flex">
          <div class="weui-cell__hd">
            <div class="check-model">
              <label class="vweui-flex weui-check__label" for="checkAll">
                <div class="weui-cell__hd">
                    <input 
                      type="checkbox" 
                      class="weui-check" 
                      name="checkAll" 
                      id="checkAll"
                      v-model="check_all"
                    />
                    <i class="weui-icon-checked"></i>
                </div>
                <div class="weui-cell__bd">
                    <p>全选</p>
                </div>
              </label>
            </div>
          </div>
          <div class="weui-cell__bd">
            <div class="invoice-price">
              合计：
              <span class="price">{{total_price | currency('¥')}}</span>
            </div>
          </div>
        </div>
        <div class="offset">
          <a href="javascript:;" class="invoice-button" @click.stop="invoice">下一步</a>
        </div>
      </div>

    </div>
    
    <!-- <transition name="slide-left">
      <router-view></router-view>
    </transition> -->

  </div>
</template>
<script>

import orderInfo from '@/components/myOrder/OrderInfo.vue';

import { 
  invoiceOrder
} from "@/utils/http/api";

export default {
  name: "Invoice",
  components: {
    orderInfo,
  },
  data() {
    return {
      userId:this.$store.state.userId,

      loading:true,
      //滚动加载参数
      distance:30, //距离底部距离多少开始加载
      isnodata:false, //是否没有数据
      infiniteMsgShow:true, //是否显示加载信息
      scrollDisabled: true, //是否开启无限滚动加载

      currPage: 1, // 当前页面
      pageSize: 0, // 每页数量
      totalCount: 0, // 所有数据数量
      totalPage: 0, // 总共几页 

      //发票管理
      orders_invoice: [],
      checked_order: [], //选中订单的集合
      check_all: false,  //是否全选
      total_price: 0,    // 发票总价格
      invoiceDetails: [], // 发票详情产品信息

    };
  },
  methods:{
    loadMore(){

      this.scrollDisabled = true;

      setTimeout(() => {
        this.currPage++; // 增加页数
        this.ordersInvoice();
      }, 500);
    },
    //发票管理 未开票的订单列表
    ordersInvoice(){

      invoiceOrder({
        userId: this.userId,
        limit: "10",
        page:this.currPage + "",
        
      }).then(res=>{
        
        
        if(res.code === 0){

          this.orders_invoice = this.orders_invoice.concat(res.page.list);
          this.currPage = res.page.currPage;
          this.totalPage = res.page.totalPage;
          this.totalCount = res.page.totalCount;

          if(this.totalCount){
            //最后一页
            if(this.currPage >= this.totalPage){
              this.scrollDisabled = true; // 将无限滚动关闭
              //this.currPage = 0;
              this.infiniteMsgShow = false; // 没有更多信息

            }else{
              this.scrollDisabled = false; // 将无限滚动给打开
              this.infiniteMsgShow = true; // 加载跟多
            }
            this.isnodata = false;
          }else{
            this.isnodata = true;
          }

          this.loading = false;

        }else{
          this.$weui.alert(res.msg);
        }

      });
    },
    checkedOrder(){
      // console.log(this.checked_order);
      if (this.checked_order.length < this.orders_invoice.length) {
        this.check_all = false;
      } else if (this.checked_order.length === this.orders_invoice.length) {
        this.check_all = true;
      }
      this.total_price = 0;
      this.invoiceDetails = [];

      if(this.checked_order.length){
        for(let i of this.checked_order){
          for(let x = 0; x < this.orders_invoice.length; x++){
            if(i === this.orders_invoice[x].orderNo){

              this.invoiceDetails.push(this.orders_invoice[x]);
              this.total_price += this.orders_invoice[x].payMoney;

              x = this.orders_invoice.length; 
			  

            }
          }
        }
      }

    },
    invoice(){
      if (this.checked_order.length) {
        this.$store.commit("setInvoiceDetails", this.invoiceDetails);
        this.$router.push({
          path: "/invoice_edit",
          query: {
            orderNo: this.checked_order,
            total: this.total_price
          }
        });
      }else{
        this.$weui.alert('请选择要开票的订单');
      }
    }
  },
  mounted(){
    this.ordersInvoice();
  },
  watch:{
    check_all(){
      if(this.check_all){
        this.checked_order = [];
        for (let i in this.orders_invoice) {
          this.checked_order.push(this.orders_invoice[i].orderNo);
        }
      }else if(this.orders_invoice.length === this.checked_order.length){
        this.checked_order = [];
      }
      this.checkedOrder();
    }
  }
};
</script>

<style lang="scss" scoped>
  .common-wrapper{
    padding-bottom: 100px;
  }

</style>