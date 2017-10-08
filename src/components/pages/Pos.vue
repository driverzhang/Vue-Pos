<template lang="html">
  <div class="pos">
    <div>
      <el-row>
        <el-col :span='7' id="order-list" class="pos-order">
          <el-tabs>
            <el-tab-pane label="点餐">
              <!--data是用来绑定数据源的， border代表表格有边框效果-->
              <el-table :data="tableData" border show-summary style="width: 100%">

                <el-table-column prop="goodsName" label="商品"></el-table-column>
                <el-table-column prop="count" label="量" width="50"></el-table-column>
                <el-table-column prop="price" label="金额" width="70"></el-table-column>
                <el-table-column label="操作" witdh="100" fixed="right">
                  <template scope="scope">
                    <el-button type="text" size="small" @click="delSingleGoods(scope.row)">
                      删除
                    </el-button>
                    <el-button type="text" size="small" @click="addOrderList(scope.row)">
                      增加
                    </el-button>
                  </template>
                </el-table-column>
              </el-table>
              <div class="totalDiv">
                <small>数量:</small>
                {{totalCount}} &nbsp;&nbsp;&nbsp;&nbsp;<small>金额:</small>
                {{totalMoney}}元
              </div>
              <div class="div-btn">
                <el-button type="warning">挂单</el-button>
                <el-button type="danger" @click="delAllGoods">删除</el-button>
                <el-button type="success" @click="checkout">结账</el-button>
              </div>
            </el-tab-pane>
            <el-tab-pane label="挂单">
              挂单
            </el-tab-pane>
            <el-tab-pane label="外卖">
              外卖
            </el-tab-pane>
          </el-tabs>
        </el-col>
        <!--商品展示-->
        <el-col :span="17">
          <div class="often-goods">
            <div class="title">常用商品</div>
            <div class="often-goods-list">

              <ul>
                <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                  <span>{{goods.goodsName}}</span>
                  <span class="o-price">￥{{goods.price}}元</span>
                </li>

              </ul>
            </div>
          </div>
          <div class="goods-type">
            <el-tabs>
              <el-tab-pane label="汉堡">
                <div>
                  <ul class='cookList'>
                    <li v-for="goods in type0Goods" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg"
                                                 width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{ goods.price }}元</span>
                    </li>
                  </ul>
                </div>
              </el-tab-pane>
              <el-tab-pane label="小食">
                <div>
                  <ul class='cookList'>
                    <li v-for="goods in type1Goods" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg"
                                                 width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{ goods.price }}元</span>
                    </li>
                  </ul>
                </div>
              </el-tab-pane>
              <el-tab-pane label="饮料">
                <div>
                  <ul class='cookList'>
                    <li v-for="goods in type2Goods" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg"
                                                 width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{ goods.price }}元</span>
                    </li>
                  </ul>
                </div>
              </el-tab-pane>
              <el-tab-pane label="套餐">
                <div>
                  <ul class='cookList'>
                    <li v-for="goods in type3Goods" @click="addOrderList(goods)">
                      <span class="foodImg"><img :src="goods.goodsImg"
                                                 width="100%"></span>
                      <span class="foodName">{{goods.goodsName}}</span>
                      <span class="foodPrice">￥{{ goods.price }}元</span>
                    </li>
                  </ul>
                </div>
              </el-tab-pane>
            </el-tabs>
          </div>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
  import axios from 'axios';

  export default {
    name: 'Pos',
    data() {
      return {
        tableData: [],
        oftenGoods: [],
        type0Goods: [],
        type1Goods: [],
        type2Goods: [],
        type3Goods: [],
        totalCount: 0,
        totalMoney: 0
      }

    },
    // 在实例创建完成后被立即调用。
    created: function () {
      axios.get('http://jspang.com/DemoApi/oftenGoods.php')
        .then(response => {
          console.log(response);
          this.oftenGoods = response.data;
        })
        .catch(error => {
          console.log(error);
          alert('网络错误，不能访问');
        })
      axios.get('http://jspang.com/DemoApi/typeGoods.php')
        .then(response => {
          console.log(response);
          this.type0Goods = response.data[0];
          this.type1Goods = response.data[1];
          this.type2Goods = response.data[2];
          this.type3Goods = response.data[3];
        })
        .catch(error => {
          console.log(error);
          alert('网络错误，不能访问');
        })
    },

    // 重点理解的业务代码！！！
    // 点击右侧的商品，然后把商品添加到左边的列表里。
    methods: {
      // 添加订单列表方法：
      addOrderList(goods) {
        // 每次添加进行清零，避免重复添加：
        this.totalCount = 0;
        this.totalMoney = 0;

        // 判断这个商品是否已经存在于列表当中：
        let isHave = false;
        for (let i = 0; i < this.tableData.length; i++) {
          console.log(this.tableData[i].goodsId);
          if (this.tableData[i].goodsId === goods.goodsId) {
            isHave = true;
          }
        }

        // 根据isHave的值判断订单列表中是否已经有此商品：
        if (isHave) {
          // 存在 就进行数量添加
          // 进行数据去重筛选：
          // filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。
          let arr = this.tableData.filter(o => o.goodsId === goods.goodsId);
          arr[0].count++;
//          console.log(arr);
        } else {
          // 不存在就推入新建的数组中：
          // 定义新建的数组：
          let newGoods = {
            goodsId: goods.goodsId,
            goodsName: goods.goodsName,
            price: goods.price,
            count: 1
          };
          // 推入新建数组中：
          this.tableData.push(newGoods);
        }

        //计算汇总金额和数量：
        this.getAllMoney();


      },
      // 删除单个商品：
      delSingleGoods(goods) {
        console.log(goods);
        this.tableData = this.tableData.filter(o => o.goodsId !== goods.goodsId);
        this.getAllMoney();
      },
      // 删除所有商品：
      delAllGoods() {
        this.tableData = [];
        this.totalMoney = 0;
        this.totalCount = 0;
      },
      // 模拟结账(返回状态)：
      // 1、设置我们Aixos的Pos方法。
      // 2、接受返回值进行处理。
      // 3、如果成功，清空现有构造器里的tableData，totalMoney，totalCount数据。
      // 4、进行用户的友好提示。这里我们只模拟3和4步:
      checkout() {
        if (this.totalCount !== 0) {
          this.tableData = [];
          this.totalCount = 0;
          this.totalMoney = 0;
          this.$message({
            message: '结账成功，感谢您的光临！',
            type: 'success'
          });
        } else {
          this.$message.error('不能空结，老板知道你心急~');
        }
      },

      // 汇总数量 和 金额：
      getAllMoney() {
        this.totalCount = 0;
        this.totalMoney = 0;
        if (this.tableData) {
          //计算汇总金额和数量：
          this.tableData.forEach((element) => {
            this.totalCount += element.count;
            this.totalMoney = this.totalMoney + (element.price * element.count);
          })
        }
      }
    },

    // el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。
    mounted: function () {
      let orderHeight = document.body.clientHeight;
      console.log(orderHeight);
      document.getElementById("order-list").style.height = orderHeight + 'px';
    }
  }
</script>

<style lang="css">
  .pos-order {
    background-color: #F9fafc;
    border-right: 1px solid #C0C0DA;
  }

  .div-btn {
    padding: 10px;
  }

  .title {
    height: 20px;
    border-bottom: 1px solid #D3DCE6;
    background-color: #F9FAFC;
    padding: 10px;
    text-align: left;
  }

  .often-goods-list ul li {
    list-style: none;
    float: left;
    border: 1px solid #E5E9F2;
    padding: 10px;
    margin: 5px;
    background-color: #fff;
  }

  .o-price {
    color: #58B7FF;
  }

  .goods-type {
    clear: both;
    cursor: pointer;
  }

  .cookList li {
    list-style: none;
    width: 25%;
    border: 1px solid #E5E9F2;
    height: auot;
    overflow: hidden;
    background-color: #fff;
    padding: 2px;
    float: left;
    margin: 3px;
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
    overflow: hidden;
    border-bottom: 1px solid #D3DCE6;
    color: red;
  }

  small {
    color: #808080;
  }
</style>
