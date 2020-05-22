<!-- 搜索页 -->

<template>
  <div>
    <div class="search">
        <div class="back" @click="goBack">
            <i class="icon-arrow_lift" ></i>
        </div>
        <form class="search-wrapper">
            <span class="search-icon" @click="search(context)"></span>
            <input 
                class="search-bar" 
                type="text" 
                v-model="context" 
                @keydown.enter="search(context)"
                placeholder="搜索店内商品" />
        </form>
    </div>
    <div class="goods">
      <!-- 右侧食物列表 -->
      <div class="foods-wrapper" ref="foodsRef">
        <ul>
          <li class="foods-list foods-list-hook"  ref="foodList"  >

            <ul>
              <li class="foods-item" v-for="(food, index) in newlist" @click="toFoodDetail(food, $event)"  :key="index">
                <div class="icon">
                  <img v-lazy="food.icon">
                </div>

                <div class="content">
                  <h2 class="name">{{ food.name }}</h2>
                  <p class="desc">{{ food.description }}</p>

                  <div class="extra">
                    <span class="count">月售{{ food.sellCount }}份</span>
                    <span class="rating">好评率{{ food.rating }}%</span>
                  </div>

                  <div class="price">
                    <span class="now">￥{{ food.price }}</span>
                    <span class="old" v-show="food.oldPrice">￥{{ food.oldPrice }}</span>
                  </div>

                  <div class="control">
                    <cart-control :food="food" @drop="drop"></cart-control>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>

      <!-- 购物车 -->
      <shopcart ref="shopcartRef"
                :selectFoods="selectFoods"
                :deliveryPrice="seller.deliveryPrice"
                :minPrice="seller.minPrice"></shopcart>
    </div>

    <!-- 商品详情页 -->
    <goods-detail @drop="drop" :food="selectedFood" ref="goodsDetailRef"></goods-detail>
  </div>
</template>

<script>
import Shopcart from '@/components/base/shopcart/shopcart'
import CartControl from '@/components/base/cart-control/cart-control'
import GoodsDetail from '@/components/index/restaurant-detail/goods-detail/goods-detail'
import BScroll from 'better-scroll'
import axios from 'axios'

export default {
  components: {
    Shopcart,
    CartControl,
    GoodsDetail
  },
  data () {
    return {
      // 商品数据
      goods: [],
      newlist:[],
      context:'',
      seller: '',
      classMap: ['decrease', 'discount', 'special', 'invoice', 'guarantee'],
      // 右侧每一大项的高度区间
      // (10) [0, 1172, 1343, 1478, 1828, 2070, 2334, 2685, 3251, 4035]
      listHeight: [],
      // 计算当前滚动的 Y 值
      scrollY: 0,
      // 需要传入详情页的商品
      selectedFood: {}
    }
  },
  props: {
  },
  watch: {},
  methods: {
    // 初始化数据
    _initData () {
      axios.get('/api/goods').then(res => {
        if (res.data.code === 0) {
          this.goods = res.data.data
        }

        // DOM 渲染完成才能进行计算
        setTimeout(() => {
          // 初始化 BScroll
          this._initScroll()
          // 计算右侧每一大项的高度
          this._calcHeight()
        }, 20)
      }).catch(err => {
        console.log(err)
      })
    },
    // 初始化 BScroll
    _initScroll () {
     if(this.scroller)
      this.menuScroll = new BScroll(this.$refs.menuRef, {
        click: true
      })
      this.foodsScroll = new BScroll(this.$refs.foodsRef, {
        click: true,
        probeType: 3
      })

      this.foodsScroll.on('scroll', pos => {
        this.scrollY = Math.abs(Math.floor(pos.y))
        // console.log(this.scrollY)
      })
    },
    // 计算右侧每一大项的高度
    _calcHeight () {
      let foodList = this.$refs.foodsRef.getElementsByClassName('foods-list-hook')
      let height = 0
      this.listHeight.push(height)

      for (let i = 0; i < foodList.length; i++) {
        height += foodList[i].clientHeight
        this.listHeight.push(height)
      }
      // (10) [0, 1172, 1343, 1478, 1828, 2070, 2334, 2685, 3251, 4035]
      // console.log(this.listHeight)
    },
    // better-scroll 默认会阻止浏览器的原生 click 事件。
    // 当设置为 true，better-scroll 会派发一个 click 事件
    // 我们会给派发的 event 参数加一个私有属性 _constructed，值为 true
    selectMenu (index, event) {
      // 防止pc端触发两次点击
      if (!event._constructed) {
        return
      }

      let foodList = this.$refs.foodsRef.getElementsByClassName('foods-list-hook')
      let el = foodList[index]
      this.foodsScroll.scrollToElement(el, 100)
    },
    drop (target) {
      // 性能优化，异步异步执行下落动画
      this.$nextTick(() => {
        this.$refs.shopcartRef.drop(target)
      })
    },
    // 点击进入商品详情页
    toFoodDetail (food, event) {
      // 防止pc端触发两次点击
      if (!event._constructed) {
        return
      }
      this.selectedFood = food
      this.$refs.goodsDetailRef.show()
    },
    search (context) {
        this.newlist=[];
        //字符串方法indexOf
        this.goods.forEach((good) => {
            good.foods.forEach((food) => {
                if (food.name.indexOf(context)>=0) {
                    this.newlist.push(food);
                }
            })
      })

    },
    goBack (){
        this.$router.push({
            path:'/restaurant',
        })
    },
    filterInput(val){   
       return val.replace(/[^\a-\z\A-\Z0-9\u4E00-\u9FA5]/g,'')
    } 
  },
  filters: {},
  computed: {
    currentIndex () {
      for (let i = 0; i < this.listHeight.length; i++) {
        let h1 = this.listHeight[i]
        let h2 = this.listHeight[i + 1]

        // 落到这个区间 或者 最后一个(无 h2)
        if ((this.scrollY >= h1 && this.scrollY < h2) || !h2) {
          return i
        }
      }
      return 0
    },
    selectFoods () {
      let select = []
      // 之前一直错，可能是 this 指向问题，不用箭头函数
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            select.push(food)
          }
        })
      })
      return select
    }
 
  },
  watch: {
      context() {
        this.$nextTick(() => {
          this.context = this.filterInput(this.context)
        })
      }
  },
  created () {
    // 初始化数据
    this._initData()
  },
  mounted () {
      this.seller = this.$route.query.seller
      console.log(this.seller)
  }

}
</script>

<style lang="scss" scoped>
@import '~@/assets/scss/const.scss';
@import '~@/assets/scss/mixin.scss';

.goods {
  display: flex;
  position: absolute;
  top: 60px;
  bottom: 46px;
  width: 100%;
  background-color: #fff;
  overflow: hidden;
  .menu-wrapper {
    flex: 0 0 80px;
    width: 80px;
    background-color: #f3f5f7;
    .menu-item {
      display: table;
      width: 56px;
      height: 54px;
      line-height: 14px;
      padding: 0 12px;
      @include onepx('bottom', true);
      &.current {
        position: relative;
        margin-top: -1px;
        background-color: #fff;
        font-weight: 700;
        z-index: 10;
      }
      .text {
        font-size: 12px;
        display: table-cell;
        vertical-align: middle;
        .icon {
          display: inline-block;
          width: 12px;
          height: 12px;
          margin-right: 2px;
          background-size: 12px 12px;
          background-repeat: no-repeat;
          vertical-align: top;
          &.decrease {
            // @include bg-image('./img/decrease_3');
          }
          &.discount {
            // @include bg-image('./img/discount_3');
          }
          &.guarantee {
            // @include bg-image('./img/guarantee_3');
          }
          &.invoice {
            // @include bg-image('./img/invoice_3');
          }
          &.special {
            // @include bg-image('./img/special_3');
          }
        }
      }
    }
  }
  .foods-wrapper {
    flex: 1;
    .foods-list {
      .title {
        padding-left: 14px;
        height: 26px;
        line-height: 26px;
        border-left: 2px solid #d9dde1;
        font-size: 12px;
        color: rgb(147, 153, 159);
        background-color: #f3f5f7;
      }
      .foods-item {
        position: relative;
        display: flex;
        margin: 18px;
        padding-bottom: 18px;
        @include onepx('bottom', true);
        .icon {
          flex: 0 0 57px;
          width: 57px;
          height: 57px;
          margin-right: 10px;
          img {
            width: 57px;
            height: 57px;
          }
        }
        .content {
          flex: 1;
          .name {
            font-size: 14px;
            margin: 2px 0 8px 0;
            height: 14px;
            line-height: 14px;
            color: rgb(7, 17, 27);
          }
          .desc {
            margin-bottom: 8px;
            line-height: 14px;
            font-size: 10px;
            color: rgb(147, 153, 159);
          }
          .extra {
            margin-bottom: 8px;
            line-height: 12px;
            font-size: 0;
            color: rgb(147, 153, 159);
            .count {
              font-size: 10px;
              margin-right: 12px;
            }
            .rating {
              font-size: 10px;
            }
          }
          .price {
            font-weight: 700;
            line-height: 24px;
            .now {
              margin-right: 8px;
              font-size: 14px;
              color: rgb(240, 20, 20);
            }
            .old {
              font-size: 10px;
              color: rgb(147, 153, 159);
              text-decoration: line-through;
            }
          }
          .control {
            position: absolute;
            right: 0;
            bottom: 1px;
          }
        }
      }
    }
  }
}
.search{
    width: 100%;
    height: 60px;
    line-height: 50px;
    background-color: #fff;
}
.search-wrapper {
  width: 100%;
  height: 50px;
  /* background: pink; */
  padding: 0 50px 0 50px;
  /* 设置盒子从边框开始计算*/
  box-sizing: border-box;
}

.search-wrapper .search-icon {
  width: 28px;
  height: 50px;
  background: url(../../index/img/business/search.png) no-repeat 11px center;
  background-size: 13px 13px;
  position: absolute;
}

.search-wrapper .search-bar {
  width: 100%;
  height: 31px;
  border: 0;
  /* 设置盒子从边框开始计算*/
  box-sizing: border-box;
  background: #eeeeee;
  border-radius: 25px;
  padding-left: 28px;
  /* 去除选中时蓝色边框*/
  outline: none;
}
.back {
    width: 30px;
    height: 50px;
    line-height: 55px;
    margin-left: 10px;
    float: left;
}
</style>
