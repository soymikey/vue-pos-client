<template>
  <div class="home-left">

    <el-tabs>
      <el-tab-pane>
        <span slot="label"> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; 点餐 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; </span>
        <div id='order-details'>
          <el-table
            :data="cartList"
            style="width: 100%"
            @cell-dblclick="showDialog"
            border
            :height='tableHeight'
            @row-click='showDialog'
          >
            >
            >
            <el-table-column
              prop="nameWithSpecs"
              label="商品"
            >
              <template slot-scope="scope">
                {{ scope.row.nameWithSpecs }}
                <div v-if="scope.row.extra.length">
                  <el-tag
                    size="medium"
                    :key="index"
                    v-for="(item, index) in scope.row.extra"
                  >{{item.name }}</el-tag>
                </div>

              </template>

            </el-table-column>
            <template v-if='!fitToMobile'>
              <el-table-column
                label="价格"
                width="70"
              >
                <template slot-scope="scope">
                  <span>{{foodAndExtraPrice(scope.row)}}</span>

                </template>
              </el-table-column>
            </template>

            <el-table-column
              prop="num"
              label="数量"
              width="50"
            ></el-table-column>

            <el-table-column
              label="操作"
              width="110"
            >
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  size="small"
                  @click="addToCart(scope.row.id, scope.row.name, scope.row.price, scope.row.specs,scope.row.extra,scope.row)"
                >+</el-button>
                <el-button
                  type="danger"
                  size="small"
                  @click="removeOutCart(scope.row)"
                >-</el-button>
              </template>
            </el-table-column>
          </el-table>

        </div>
        <div class="summary">
          <div class="totalDiv">
            <small>数量：</small>

            <strong>{{totalNum}}</strong> &nbsp;&nbsp;&nbsp;&nbsp;
            <small>总计：</small>
            <strong>{{totalPrice}}</strong> 元
          </div>
          <div class="order-btn">
            <el-row>
              <el-col :span="8">
                <el-button
                  type="warning"
                  class="button"
                  :disabled='totalNum===0'
                  @click="notAvailable()"
                >挂单</el-button>
              </el-col>
              <el-col :span="8">
                <el-button
                  type="danger"
                  class="button"
                  @click="clearCart()"
                  :disabled='totalNum===0'
                >删除</el-button>
              </el-col>
              <el-col :span="8">
                <el-button
                  type="success"
                  class="button"
                  @click="checkout()"
                  :disabled='totalNum===0'
                > 结账</el-button>
              </el-col>
            </el-row>
            <!-- <el-button
              type="warning"
              class="button"
              :disabled='totalNum===0'
              @click="notAvailable()"
            >挂单</el-button>
            <el-button
              type="danger"
              class="button"
              @click="clearCart()"
              :disabled='totalNum===0'
            >删除</el-button>
            <el-button
              type="success"
              class="button"
              @click="checkout()"
              :disabled='totalNum===0'
            > 结账</el-button> -->

          </div>
        </div>

      </el-tab-pane>

      <el-tab-pane>
        <span slot="label"> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; 挂单 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; </span>
        <div>此功能还未开放</div>
      </el-tab-pane>
      <el-tab-pane label="外卖">
        <span slot="label"> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; 外卖 &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; </span>
        <div>此功能还未开放</div>

      </el-tab-pane>

    </el-tabs>
    <!-- 显示额外添加 -->
    <section
      class='extra-dialog'
      v-if="showExtra"
    >
      <el-dialog
        title='额外'
        :visible.sync="showExtra"
        width="80%"
      >
        <el-tabs type="border-card">
          <el-tab-pane
            v-bind:key="index"
            v-for="(category,index) in extraMenu "
            :label="category.name"
            class="tag-container"
          >

            <div
              v-for="(item,index) in category.extras"
              :key='index'
              class="extra-box"
              :class="{'actived':item.selected}"
              @click="selectExtra(item)"
            >
              <div
                class="extra-item"
                border
              >{{item.name}}</div>

            </div>
          </el-tab-pane>

        </el-tabs>
        <div class="button-container">
          <el-button @click="cancelExtra">取 消</el-button>
          <el-button
            type="primary"
            @click="confirmExtra"
          >确 定</el-button>
        </div>

        <!-- <el-tabs
          type="border-card"
          tab-position='left'
          style="height: 100%;"
        >
          <el-tab-pane
            v-bind:key="category.name"
            v-for="category in extraMenu "
            :label="category.name"
            class="tag-container"
          >
            <div
              v-for="(item,index) in category.children"
              :key='index'
              :label="item"
              class="tag"
            >
              <el-checkbox-button v-model="item.selected">{{item.name}}</el-checkbox-button>
            </div>

          </el-tab-pane>
        </el-tabs>
        <div class="button-container">
          <el-button @click="cancelExtra">取 消</el-button>
          <el-button
            type="primary"
            @click="confirmExtra"
          >确 定</el-button>
        </div> -->

      </el-dialog>
    </section>
  </div>
</template>

<script>
import { mapState, mapMutations } from 'vuex'
import PrintHTML from '../../config/PrintHTML.js'
import { placeOrder, getExtras } from '../../apiService/clientApi.js'

export default {
  data() {
    return {
      data: [],
      tableData: [], //订单列表的值

      totalMoney: 0, //订单总价格
      tableHeight: 0, //table的高度
      // tableHeight: window.innerHeight - 216, //table的高度
      showExtra: false,
      extraMenu: null,
      selectedFood: null,
      canvasUrl: null,
      fitToMobile: false
    }
  },
  created() {
    if (window.innerWidth < 1024) {
      this.fitToMobile = true
      this.tableHeight = window.innerHeight - 157
    } else {
      this.tableHeight = window.innerHeight - 216
    }
  },
  mounted: function() {
    this.initData()
  },
  computed: {
    ...mapState(['userInfo', 'cartList', 'restaurantInfo']),

    //购物车中总共商品的价格
    totalPrice: function() {
      let total = 0
      this.cartList.forEach(item => {
        if (item.extra) {
          let extraPirce = 0
          for (const extra of item.extra) {
            extraPirce += extra.price
          }
          total += item.num * item.price + extraPirce * item.num
        } else {
          total += item.num * item.price
        }
      })
      return total.toFixed(2)
    },

    //购物车中总共商品的数量
    totalNum: function() {
      let num = 0
      this.cartList.forEach(item => {
        num += item.num
      })
      return num
    }
  },
  watch: {},
  methods: {
    ...mapMutations(['ADD_CART', 'REDUCE_CART', 'CLEAR_CART']),
    //加入购物车，所需7个参数，商铺id，食品分类id，食品id，食品规格id，食品名字，食style  tableRowStyle({ row, rowIndex }) {
    // tableRowStyle({ row, rowIndex }) {
    //   const index = this.tableData.indexOf(this.selectedFood)

    //   if (rowIndex === index) {
    //     return { backgroundColor: '#409EFF' }
    //   }
    //   return ''
    // },

    async initData() {
      const extras = await getExtras({
        restaurantId: this.userInfo.restaurantId
      })
      let formatedExtras = []
      for (const category of extras.data) {
        for (const item of category.extras) {
          item.selected = false
        }
        formatedExtras.push(category)
      }
      this.extraMenu = formatedExtras
      // this.extraMenu = extras.data
    },

    foodAndExtraPrice(item) {
      let totalprice = 0
      if (item.extra.length) {
        let extraPrice = 0
        for (const extra of item.extra) {
          extraPrice += extra.price
        }
        totalprice = item.price + extraPrice
      } else {
        totalprice = item.price
      }

      return totalprice.toFixed(2)
    },
    addToCart(food_id, name, price, specs, extra) {
      let nameWithSpecs = specs ? `${name}+${specs}` : name
      this.ADD_CART({
        shopid: this.restaurantInfo.restaurantId,
        food_id,
        name,
        price,
        specs,
        nameWithSpecs,
        extra
      })
    },

    //移出购物车，所需7个参数，商铺id，食品分类id，食品id，食品规格id，食品名字，食品价格，食品规格
    removeOutCart(food) {
      let index = this.cartList.indexOf(food)

      this.REDUCE_CART({
        index
      })
    },

    //清除购物车
    clearCart() {
      // this.toggleCartList();
      this.$confirm('是否删除订单?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$message({
            type: 'success',
            message: '删除成功'
          })
          this.CLEAR_CART()
        })
        .catch(error => {})
    },
    showDialog(row, column, cell, event) {
      if (column.label == '商品') {
        this.selectedFood = row
        if (this.selectedFood.extra.length) {
          //把已选择的extra id 放到数组里面
          let extraId = []
          for (const extra of this.selectedFood.extra) {
            extraId.push(extra.id)
          }

          for (const category of this.extraMenu) {
            for (const item of category.extras) {
              if (extraId.includes(item.extraId)) {
                item.selected = true
              } else {
                item.selected = false
              }
            }
          }
        } else {
          for (const category of this.extraMenu) {
            for (const item of category.extras) {
              item.selected = false
            }
          }
        }

        this.showExtra = !this.showExtra
      }
    },

    cancelExtra() {
      this.showExtra = !this.showExtra
    },
    confirmExtra() {
      const selectedExtraList = []
      for (const category of this.extraMenu) {
        for (const item of category.extras) {
          if (item.selected) {
            selectedExtraList.push({
              name: category.name + item.name,
              id: item.extraId,
              price: item.price
            })
          }
        }
      }
      let food_id = this.selectedFood.id
      let name = this.selectedFood.name
      let price = this.selectedFood.price
      let specs = this.selectedFood.specs
      let nameWithSpecs = this.selectedFood.nameWithSpecs
      let extra = selectedExtraList
      let food = this.selectedFood

      this.ADD_CART({
        shopid: this.restaurantInfo.restaurantId,
        food_id,
        name,
        price,
        specs,
        nameWithSpecs,
        extra,
        food
      })

      this.showExtra = !this.showExtra
    },
    async checkout() {
      if (this.cartList.length) {
        // PrintHTML(this.cartList, this.totalNum, this.totalPrice)
        const param = {
          userName: this.userInfo.username,
          userId: this.userInfo.userId,
          restaurantId: this.restaurantInfo.restaurantId,
          restaurantName: this.restaurantInfo.name,
          cartId: new Date().valueOf(),
          totalPrice: this.totalPrice,
          totalQuantity: this.totalNum,
          entities: this.cartList,
          address: 123,
          description: '测试订单'
        }

        const result = await placeOrder(param)
        this.CLEAR_CART()
        this.$message({
          type: 'success',
          message: result.message
        })
      } else {
        window.alert('没有需要打印的')
      }
    },
    selectExtra(item) {
      item.selected = !item.selected //!item.selected
    },
    notAvailable() {
      this.$message({
        type: 'error',
        message: '此功能暂时还未开放..'
      })
    }
  }
}
</script>

<style scoped  lang="scss">
.order-btn {
  margin-top: 20px;
  margin-bottom: 20px;
  text-align: center;
}

.totalDiv {
  height: auot;
  overflow: hidden;
  text-align: right;
  font-size: 16px;
  background-color: #fff;
  border-bottom: 1px solid #e5e9f2;
  padding: 10px;
}
.button-container {
  margin-top: 30px;
}
.tag-container {
  display: flex;
  flex-wrap: wrap;
  .extra-box {
    width: 15%;
    height: 40px;
    border: 1px solid #000;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0 1px 0 1px;
  }
  .actived {
    background-color: #409eff;
  }
}
.button {
  width: 80%;
  height: 80px;
}
.el-table .warning-row {
  background: blue;
}

.el-table .success-row {
  background: #f0f9eb;
}

@media screen and (max-width: 1023px) {
  .button {
    width: 80%;
    height: 40px;
  }
  .order-btn {
    margin-top: 10px;
    margin-bottom: 10px;
    text-align: center;
  }
}

//#409EFF
</style>
