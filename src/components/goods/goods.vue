<template>
  <div class="goods">
    <div class="scroll-nav-wrapper">
      <cube-scroll-nav
        :side="true"
        :data="goods"
        :options="scrollOptions"
        v-if="goods.length"
      >
        <template slot="bar" slot-scope="props">
          <cube-scroll-nav-bar
            direction="vertical"
            :labels="props.labels"
            :txts="barTxts"
            :current="props.current"
          >
            <template slot-scope="props">
              <div class="text">
                <support-ico
                  v-if="props.txt.type>=1"
                  :size=3
                  :type="props.txt.type"
                ></support-ico>
                <span>{{props.txt.name}}</span>
                <span class="num" v-if="props.txt.count">
                  <bubble :num="props.txt.count"></bubble>
                </span>
              </div>
            </template>
          </cube-scroll-nav-bar>
        </template>
        <cube-scroll-nav-panel
          v-for="good in goods"
          :key="good.name"
          :label="good.name"
          :title="good.name"
        >
          <ul>
            <li
              @click="selectFood(food)"
              v-for="food in good.foods"
              :key="food.name"
              class="food-item"
            >
              <div class="icon">
                <img width="57" height="57" :src="food.icon">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span>
                  <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <div class="cart-control-wrapper">
                  <cart-control @add="onAdd" :food="food"></cart-control>
                </div>
              </div>
            </li>
          </ul>
        </cube-scroll-nav-panel>
      </cube-scroll-nav>
    </div>
    <div class="shop-cart-wrapper">
      <shop-cart
        ref="shopCart"
        :selectFoods="selectFoods"
        :deliveryPrice="seller.deliveryPrice"
        :minPrice="seller.minPrice">
      </shop-cart>
    </div>
  </div>
</template>

<script>
import { getGoods } from 'api/index'
import ShopCart from 'components/shop-cart/shop-cart'
import CartControl from 'components/cart-control/cart-control'
import SupportIco from 'components/support-ico/support-ico'
import Bubble from 'components/bubble/bubble'

export default {
  name: 'goods',
  props: {
    data: {
      type: Object,
      default() {
        return {}
      }
    }
  },
  data() {
    return {
      goods: [],
      selectedFood: {},
      scrollOptions: {
        click: false,
        directionLockThreshold: 0
      }
    }
  },
  computed: {
    seller() {
      return this.data.seller
    },
    selectFoods() {
      const ret = []
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            ret.push(food)
          }
        })
      })
      return ret
    },
    barTxts() {
      const ret = []
      this.goods.forEach((good) => {
        const { type, name, foods } = good
        let count = 0
        foods.forEach((food) => {
          count += food.count || 0
        })
        ret.push({
          type,
          name,
          count
        })
      })
      return ret
    }
  },
  methods: {
    selectFood(food) {
      this.selectedFood = food
      this._showFood()
      this._showShopCartSticky()
    },
    fetch() {
      if (!this.fetched) {
        this.fetched = true
        getGoods().then((goods) => {
          this.goods = goods
        })
      }
    },
    onAdd(el) {
      this.$refs.shopCart.drop(el)
    },
    _showFood() {
      this.foodComp = this.foodComp || this.$createFood({
        $props: {
          food: 'selectedFood'
        },
        $events: {
          leave: () => {
            this._hideShopCartList()
          },
          add: (el) => {
            this.shopCartStickyComp.drop(el)
          }
        }
      })
      this.foodComp.show()
    },
    _showShopCartSticky() {
      this.shopCartStickyComp = this.shopCartStickyComp || this.$createShopCartSticky({
        $props: {
          selectFoods: 'selectFoods',
          deliveryPrice: this.seller.deliveryPrice,
          minPrice: this.seller.minPrice,
          fold: true
        }
      })
      this.shopCartStickyComp.show()
    },
    _hideShopCartList() {
      this.shopCartStickyComp.hide()
    }
  },
  components: {
    ShopCart,
    CartControl,
    SupportIco,
    Bubble
  }
}
</script>

<style lang="stylus" scoped>
  @import "~common/stylus/mixin"
  @import "~common/stylus/variable"

  .goods
    position: relative
    text-align: left
    height: 100%
    .scroll-nav-wrapper
      position: absolute
      width: 100%
      top:0
      left: 0
      bottom: 48px
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        position: relative
        .icon
          -webkit-box-flex: 0
          flex: 0 0 57px
          margin-right: 10px
          img
            height: auto
        .content
          -webkit-box-flex: 1
          flex: 1
          .name
            color: #333
            font-size: 14px
            height: 14px
            line-height: 14px
            margin: 2px 0 8px
          .desc
            line-height: 12px
            margin-bottom: 8px
            color: #999
            font-size: 10px
          .extra
            color: #999
            font-size: 10px
            line-height: 10px
            .count
              margin-right: 12px
          .price
            font-weight: 700
            line-height: 24px
            .now
              color: #f01414
              font-size: 14px
              margin-right: 8px
            .old
              color: #999
              font-size: 10px
              text-decoration: line-through
          .cart-control-wrapper
            bottom: 12px
            position: absolute
            right: 0
    >>> .cube-scroll-nav-bar
      width: 80px
      white-space: normal
      overflow: hidden
    >>> .cube-scroll-nav-bar-item
      padding:0 10px
      display: flex
      align-items: center
      text-align: center
      height: 56px
      line-height: 14px
      font-size: $fontsize-small
      background: $color-background-ssss
      .bubble
        background: linear-gradient(90deg,#fc9153,#f01414)
        border-radius: 16px
        color: #fff
        display: inline-block
        font-family: Helvetica
        font-size: 10px
        font-weight: 700
        height: 16px
        line-height: 16px
        padding: 0 5px
        text-align: center
      .text
        -webkit-box-flex: 1
        flex: 1
        position: relative
      .num
        position: absolute
        right: -8px
        top: -10px
      .support-icon
        display : inline-block
        vertical-align: top
        margin-right: 4px
    >>> .cube-scroll-nav-bar-item_active
      background: #fff
      color: #333
    >>> .cube-scroll-wrapper
      position: relative
      height: 100%
      overflow: hidden
      flex: 1
    >>> .cube-scroll-nav-panel-title
      background: $color-background-ssss
      border-left: 2px solid #d9dde1
      color: #666
      font-size: 12px
      height: 26px
      line-height: 26px
      padding-left: 14px
    .shop-cart-wrapper
      bottom: 0
      height: 48px
      left: 0
      position: absolute
      width: 100%
      z-index: 50
</style>
