<template>
  <div>
          <div class="box ">
       <div class="box_start ">
         <div class="f20">我的订单</div>
       </div> 
       <div class="mt-30">
         <div class="dd-nav">
        <a href="JavaScript:;"  class="" v-bind:class=" listType===''?'act':''" v-on:click="_getOrderList('',1)">全部订单</a>
        <a href="JavaScript:;"  class="" v-bind:class=" listType==='wait_pay'?'act':''" v-on:click="_getOrderList('wait_pay',1)">待付款</a>
        <a href="JavaScript:;"  class="" v-bind:class=" listType==='wait_send'?'act':''" v-on:click="_getOrderList('wait_send',1)">待发货</a>
        <a href="JavaScript:;"  class="" v-bind:class=" listType==='wait_receive'?'act':''" v-on:click="_getOrderList('wait_receive',1)">待收货</a>
        <a href="JavaScript:;"  class="" v-bind:class=" listType==='wait_assessment'?'act':''" v-on:click="_getOrderList('wait_assessment',1)">待评价</a>
        <a href="JavaScript:;"  class="" v-bind:class=" listType==='end'?'act':''" v-on:click="_getOrderList('end',1)">已完成</a>
        <!-- <div class="btn"><input type="text"  placeholder="订单编号" class="">
          <i class="fa fa-search fa-lg"></i></div> -->
      </div>
       </div>
       <div class="ddlist-nav mt-60 mb-30">
        <ul>
          <li>商品名</li>
          <li>订单金额</li>
          <li>订单状态</li>
          <li>操作</li>
        </ul>
      </div>
      <div class="ddlist-box">
        <div class="ddlist " v-for="item in orderList " >
          <div class="tit"><span >下单时间：</span><span >订单号：{{item.order_no}}</span><span >{{item.merchants_name}}
            <a  href="http://kefu.easemob.com/webim/im.html?tenantId=45997"><img src="@/assets/icon/lianxishangjia.png"></a></span></div>
          <div class="table">
            <table cellspacing="0" cellpadding="0">
              <tbody><tr>
                <td class="td1" style="padding: 0;">
                  <div class="dd-l-list clearfix " v-for="citem in item.orderBeans ">
                    <ul>
                      <li class="img"><router-link :to="{path: '/mall/goodsDetails', query: {goods_id: citem.goods_id}}"><img v-bind:src="citem.goods_img" ></router-link></li>
                      <li class="txt"><router-link :to="{path: '/mall/goodsDetails', query: {goods_id: citem.goods_id}}" v-text="citem.goods_name" ></router-link ></li>
                      <li class="num ">×{{citem.goods_num}}</li>

                      <li class="sqsh" >
                        <!-- <span class="blue" ><a href="#">申请退款</a></span> -->
                        <!-- <span class="blue" >等待审核</span> -->
                        <!-- <span class="blue" >退款成功</span> -->
                        <!-- <span class="blue" >退款申请已拒绝</span> -->
                      </li>
                    </ul>
                  </div>
                
                </td>
                <td class="td2 red ">￥{{item.order_actual_price}}</td>
                <td class="td3"><span class="">{{item.order_state==='cancel'?'取消':item.order_state==='wait_pay'?'待付款':item.order_state==='wait_send'?'待发货':item.order_state==='wait_receive'?'待确认收货':item.order_state==='wait_assessment'?'待评价':item.order_state==='end'?'已结束':''}}</span>
                  <router-link :to="{path: '/portal/orderDetail',query: {order_id:item.order_id,order_merchants_id:item.order_merchants_id}}">订单详情</router-link>
                </td>
                <td class="td4 " >
                  <span v-if="item.order_state==='wait_pay'" class="huise"></span><router-link :to="{path: '/orderPay',query: {order_id:item.order_id,order_no:item.order_no}}" class="btn" >立即付款</router-link>
                  <span class="btn" v-if="item.order_state==='wait_receive'" v-on:click="_receiveOrder(item.order_merchants_id)">确认收货</span>
                  <span class="btn" v-if="item.order_state==='end'" v-on:click="_deleteOrder(item.order_merchants_id)">删除订单</span>
                   <span class="del" v-if="item.order_state==='wait_pay'" v-on:click="_cancelOrder(item.order_merchants_id)"> 取消订单</span>
                </td>
              </tr>
            </tbody></table>
          </div>
        </div>
         <div class="paginaction" v-show="allPage >= 1">
            <el-pagination
              :prev-text="'上一页'"
              :next-text="'下一页'"
              background
              @current-change="handleCurrentChange"
              :current-page.sync="currentPage"
              :page-size="4"
              layout="total,prev, pager, next, jumper"
              :page-count="allPage">
            </el-pagination>
          </div>  
      </div>

       
       
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import '@/assets/css/cuimeng_style.css'
import { Pagination } from 'element-ui'
import storage from '@/common/storage'
import api from '@/api/order'
export default {
  data () {
    return {
      orderList : [],
      currentPage: 1, // 当前页
      allPage: 1 ,// 总页数
      sortType: 1, // 排序类型默认为1(综合)
      listType:''
    }
  },
  computed: {
    ...mapGetters([
      'orderType'
    ])
  },
  methods: {
   
    // 获取订单列表
    _getOrderList (type,p) {
      this.orderList = []
      this.currentPage = p || 1
      this.listType = type || '' //cancel：取消 wait_pay:待付款 wait_send:带发货 wait_receive：待确认收货 wait_assessment：待评价 end：已结束 
      api.getOrderList({
        uid: this.getCookie('uid'),
        token: this.getCookie('token'),
        order_state : this.listType,
        p : this.currentPage,
        pagesize: 4
      }).then(res => {
         console.log(res)
        if(res.status === 'ok'){
          this.allPage = res.data.page
          for(let i=0; i<res.data.list.length; i++){
            this.orderList.push(res.data.list[i])
          }
        }else if(res.status === 'error'){
          this.promptFun({
            content: res.data,
            type: 'error'
          })
        }
      })
    },
    // 取消订单
    _cancelOrder (orid) {
      api.cancelOrder({
        uid: this.getCookie('uid'),
        token: this.getCookie('token'),
        order_merchants_id : orid
      }).then(res => {    
      if(res.status === 'ok'){     
        this.promptFun({
            content: res.data,
            type: 'true'
          })
        this._getOrderList();
        }else if(res.status === 'error'){
          this.promptFun({
            content: res.data,
            type: 'error'
          })
        }
      })
    },
    // 确认收货订单
    _receiveOrder (orid) {
      api.receiveOrder({
        uid: this.getCookie('uid'),
        token: this.getCookie('token'),
        order_merchants_id : orid
      }).then(res => {    
      if(res.status === 'ok'){     
        this.promptFun({
            content: res.data,
            type: 'true'
          })
        this._getOrderList();
        }else if(res.status === 'error'){
          this.promptFun({
            content: res.data,
            type: 'error'
          })
        }
      })
    },
    // 删除订单
    _deleteOrder (orid) {
      api.deleteOrder({
        uid: this.getCookie('uid'),
        token: this.getCookie('token'),
        order_merchants_id : orid
      }).then(res => {    
      if(res.status === 'ok'){     
        this.promptFun({
            content: res.data,
            type: 'true'
          })
        this._getOrderList();
        }else if(res.status === 'error'){
          this.promptFun({
            content: res.data,
            type: 'error'
          })
        }
      })
    },
     //onPage
    onPage (count) {
      this.currentPage = count
    },
     // 页数改变事件
    handleCurrentChange (val) {
      this._getUserGoodsCollection(this.listType , val)
    },

  },
  created () {
    this._getOrderList(this.orderType)
  },
  components: {
    ElPagination: Pagination,
  }
}
</script>

<style scoped>
 .box{
    background-color: #fff;
    padding: 30px;
  }

  dd-nav>a.act, .dd-nav>a:hover {
    border-bottom: 2px solid #3f8fdf;
    color: #3f8fdf;
}
 .act {
    border-bottom: 2px solid #3f8fdf;
    color: #3f8fdf;
}
.dd-nav>a {
    display: inline-block;
    border-bottom: 2px solid #fff;
    margin-right: 10px;
    line-height: 38px;
}
.dd-nav>div {
    float: right;
    display: inline-block;
    width: 200px;
    height: 40px;
    border: 1px solid #ebebeb;
    background: #f1f1f1;
    line-height: 38px;
}
.dd-nav>div input {
    border: 0px;
    padding-left: 7px;
    height: 38px;
    line-height: 38px;
    width: 140px;
    background: #f1f1f1;
    float: left;
}
input {
    outline: none;
    border-radius: 0px;
    font-family: tahoma,arial,Microsoft YaHei,Hiragino Sans GB,"\u5b8b\u4f53",sans-serif;
    font-size: 14px;
}
.dd-nav>div i {
    width: 58px;
    border: 0px;
    height: 38px;
    text-align: center;
    color: #fff;
}
.ddlist-nav {
    width: 100%;
    height: 50px;
    line-height: 50px;
    background: #f1f2f6;
}
.ddlist-nav ul li:nth-child(1) {
    width: 441px;
}
.ddlist-nav ul li {
    text-align: center;
    float: left;
    width: 145px;
}
.dd-l-list .sqsh{width:80px;text-align:center}
.ddlist {
    margin-bottom: 40px;
}
.ddlist>.tit {
    height: 36px;
    line-height: 36px;
    background: #fbfbfb;
}
.ddlist>.tit span {
    margin-left: 30px;
    font-size: 12px;
    color: #666;
}
.ddlist>.tit span img {
    width: 14px;
    height: 14px;
    margin-left: 5px;
}
.table table {
    border-collapse: collapse;
    width: 100%;
}
.ddlist-box tr {
    border: 1px solid #ebebeb;
}
.ddlist-box .td1 {
    width: 436px;
}
.ddlist-box td {
    border: 1px solid #ebebeb;
    padding: 30px 20px;
}
.ddlist-box .td2 {
    width: 145px;
}
.dd-l-list:last-child {
    margin-bottom: 0px!important;
}
.dd-l-list:first-child {
    border: none;
}
.dd-l-list {
    margin-bottom: 0px;
    border-top: 1px solid #ebebeb;
}
.dd-l-list .img {
    width: 80px;
    height: 80px;
    overflow: hidden;
    display: table-cell;
    text-align: center;
    vertical-align: middle;
}
.dd-l-list .img img {
    width: 80px;
}
.dd-l-list .txt {
    width: 190px;
    padding-left: 20px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}
.dd-l-list .num {
    width: 40px;
    text-align: center;
}
.dd-l-list li {
    display: inline-block;
    float: left;
    line-height: 80px;
}
.red {
    color: #fd655a!important;
}
.blue{color:#2173c5!important};
.ddlist-box .td3 {
    width: 145px;
}
.ddlist-box td {
    border: 1px solid #ebebeb;
    padding: 30px 20px;
}
.ddlist-box .td3 span {
    color: #ff9e31;
}
.ddlist-box .td3 a {
    display: block;
}
.ddlist-box .td4 {
    position: relative;
}
.ddlist-box .td4 {
    width: 145px;
}
.ddlist-box .td4 span {
    margin-bottom: 8px;
    cursor: pointer;
    color: #999;
}
.ddlist-box .td4 .btn {
    display: block;
    padding: 3px 6px;
    border-radius: 5px;
    color: #3f8fdf;
    border: 1px solid #3f8fdf;
    text-align: center;
}
.ddlist-box .td4 span {
    margin-bottom: 8px;
    cursor: pointer;
    color: #999;
}
.ddlist-box .td4 .del {
    display: block;
    padding: 3px 6px;  
    text-align: center;
}
.paginaction{
    margin: 10px auto 0;
    width: 80%;
  }
</style>
