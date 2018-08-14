<template>
  <div class="pos">
    <el-row>
      <el-col  :span='7' class="pos-order"  id="order-list">
        <el-tabs>
            <el-tab-pane label ="点餐">
                <el-table :data="tableData" border style="width = 100%  ">
                    <el-table-column prop='goodsName' label = '商品名称' ></el-table-column>
                    <el-table-column prop='count' label = '数量'></el-table-column>
                    <el-table-column prop='price' label = '金额'> </el-table-column>
                    <el-table-column  label = '操作'  fixed="right">
                        <template slot-scope="scope">
                            <el-button type = 'text' size = "small" @click="delSingleGoods(scope.row)" >删除</el-button>
                            <el-button type = 'text' size = 'small' @click="addOrderList(scope.row)"> 增加 </el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <div class="totalDiv">
                   <small> 数量 ：</small> {{totalCount}} &nbsp;&nbsp;&nbsp; <small> 金额 ：</small> {{totalMoney}}元
                </div>
                <div class="div-btn">
                    <el-button type="warning" size="small">挂单</el-button>
                    <el-button type="danger" size="small" @click="delAllGoods()">删除</el-button>
                    <el-button type="success" size="small" @click="checkout()">结账</el-button>
                </div>
            </el-tab-pane>
            <el-tab-pane label ="挂单">挂单</el-tab-pane>
            <el-tab-pane label ="外卖">外卖</el-tab-pane>
        </el-tabs>
      

      </el-col>
      <el-col :span="17" >
        <div class="often-goods">
          <div class="tittle">常用商品 </div>
            <div class="often-goods-list">
              <ul>
                <li v-for="(goods,index) in oftenGoods" :key ='index' @click="addOrderList(goods)">
                  <span>{{goods.goodsName}}</span>
                  <span class="o-price">￥{{goods.price}}元</span>
                </li>
              </ul>
          </div>
        </div>
        <div class="goods-type">
          <el-tabs>
          <el-tab-pane label = '汉堡'>
           <div>
             <ul class="cookList">
              <li v-for="(goods,index) in type0Goods" :key="index" @click="addOrderList(goods)">
                 <span class="foodImg"><img :src = 'goods.goodsImg' width="100%"> </span>
                 <span class="foodName">{{goods.goodsName}}</span>
                 <span class="foodPrice">￥{{goods.price}}元</span>
              </li>
             </ul>
           </div>
            </el-tab-pane>
          <el-tab-pane label = '小食'>
              <ul class="cookList">
              <li v-for="(goods,index) in type1Goods" :key="index" @click="addOrderList(goods)">
                 <span class="foodImg"><img :src = 'goods.goodsImg' width="100%"> </span>
                 <span class="foodName">{{goods.goodsName}}</span>
                 <span class="foodPrice">￥{{goods.price}}元</span>
              </li>
             </ul>
          </el-tab-pane>
          <el-tab-pane label = '饮料'>
              <ul class="cookList">
              <li v-for="(goods,index) in type2Goods" :key="index" @click="addOrderList(goods)">
                 <span class="foodImg"><img :src = 'goods.goodsImg' width="100%"> </span>
                 <span class="foodName">{{goods.goodsName}}</span>
                 <span class="foodPrice">￥{{goods.price}}元</span>
              </li>
             </ul>
          </el-tab-pane>
          <el-tab-pane label = '套餐'>
              <ul class="cookList">
              <li v-for="(goods,index) in type3Goods" :key="index" @click="addOrderList(goods)">
                 <span class="foodImg"><img :src = 'goods.goodsImg' width="100%"> </span>
                 <span class="foodName">{{goods.goodsName}}</span>
                 <span class="foodPrice">￥{{goods.price}}元</span>
              </li>
             </ul>
          </el-tab-pane>
        </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "pos",
  data() {
    return {
      tableData: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0
    };
  },
  created: function() {
    axios
      .get("http://jspang.com/DemoApi/oftenGoods.php")
      .then(Response => {
        console.log(Response);
        this.oftenGoods = Response.data;
      })
      .catch(Err => {
        alert("网络错误，不能访问");
      });

    axios
      .get("http://jspang.com/DemoApi/typeGoods.php")
      .then(Response => {
        console.log(Response);
        this.type0Goods = Response.data[0];
        this.type1Goods = Response.data[1];
        this.type2Goods = Response.data[2];
        this.type3Goods = Response.data[3];
      })
      .catch(Err => {
        alert("网络错误，不能访问");
      });
  },
  mounted: function() {
    var orderHeight = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderHeight + "px";
  },
  methods: {
    addOrderList(goods) {
      this.totalMoney = 0;
      this.totalCount = 0;
      // 商品是否已经存在于订单列表中
      let isHave = false;
      for (let i = 0; i < this.tableData.length; i++) {
        if (this.tableData[i].goodsId == goods.goodsId) {
          isHave = true;
        }
      }

      // 根据判断的值编写业务逻辑
      if (isHave) {
        // 改变列表中的商品数量
        let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
        arr[0].count++;
      } else {
        let newGoods = {
          goodsId: goods.goodsId,
          goodsName: goods.goodsName,
          price: goods.price,
          count: 1
        };
        this.tableData.push(newGoods);
      }
      this.getAllMoney();
    },
    // 删除单个商品
    delSingleGoods(goods) {
      this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId);
      this.getAllMoney();
    },
    delAllGoods(){
        this.tableData= [];
        this.totalMoney = 0;
        this.totalCount = 0;
    },
    getAllMoney() {
      this.totalCount = 0;
      this.totalMoney = 0;
      if (this.tableData) {
        this.tableData.forEach(element => {
          this.totalCount += element.count;
          this.totalMoney += element.price * element.count;
        });
      }
    },
    // 模拟结账
    checkout(){
        if(this.totalCount!=0){
            this.totalMoney = 0;
            this.totalCount = 0;
            this.tableData = [];
            this.$message({
                message:'结账成功，欢迎下次光临',
                type:'success'
            })

        }else{
            this.$message.error('不能空结哦~')
        }

    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pos-order {
  background-color: white;
  border-right: 1px solid #d3dce6;
}
.div-btn {
  margin: 10px;
}
.tittle {
  height: 38px;
  border-bottom: 2px solid #d3dce6;
  background-color: #f9fafc;
  line-height: 38px;
  text-align: left;
  padding-left: 10px;
}
.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #f9fafc;
  padding: 10px;
  margin: 10px;
  background-color: #ffffff;
  cursor: pointer;
}
.o-price {
  color: #58b7ff;
}
.goods-type {
  clear: both;
}
.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #e5e9f2;
  height: auot;
  overflow: hidden;
  background-color: #fff;
  padding: 12px;
  float: left;
  margin: 2px;
  cursor: pointer;
}
.cookList li span {
  display: block;
  float: left;
}
.foodImg {
  width: 40%;
}
.foodName {
  font-size: 18px;
  padding-left: 10px;
  color: brown;
}
.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}
.totalDiv {
  background-color: #fff;
  padding: 18px;
  border-bottom: 1px solid #d3dce6;
}
</style>
