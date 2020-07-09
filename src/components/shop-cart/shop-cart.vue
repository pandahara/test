<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">
              <bubble :num="totalCount"></bubble>
            </div>
          </div>
          <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
          <div class="desc">另需配送费¥{{deliveryPrice}}元</div>
        </div>
        <div class="content-right">
          <div @click="pay" class="pay" :class="payClass">
            {{payDesc}}
          </div>
        </div>
      </div>
      <div class="ball-container">
        <div v-for="(ball, index) in balls" :key="index">
          <transition
            @before-enter="beforeDrop"
            @enter="dropping"
            @after-enter="afterDrop"
          >
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Bubble from 'components/bubble/bubble'

const BALL_LEN = 10

function createBalls() {
  const ret = []
  for (let i = 0; i < BALL_LEN; i++) {
    ret.push({
      show: false
    })
  }
  return ret
}

export default {
  name: 'shop-cart',
  props: {
    selectFoods: {
      type: Array,
      default() {
        return []
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    },
    fold: {
      type: Boolean,
      default: true
    },
    sticky: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      balls: createBalls(),
      listFold: this.fold
    }
  },
  computed: {
    totalPrice() {
      let total = 0
      this.selectFoods.forEach((food) => {
        total += food.price * food.count
      })
      return total
    },
    totalCount() {
      let count = 0
      this.selectFoods.forEach((food) => {
        count += food.count
      })
      return count
    },
    payDesc() {
      if (this.totalPrice === 0) {
        return `¥${this.minPrice}元起送`
      } else if (this.totalPrice < this.minPrice) {
        const diff = this.minPrice - this.totalPrice
        return `还差¥${diff}元起送`
      } else {
        return `去结算`
      }
    },
    payClass() {
      if (!this.totalCount || this.totalPrice < this.minPrice) {
        return 'not-enough'
      } else {
        return 'enough'
      }
    }
  },
  created() {
    this.dropBalls = []
  },
  methods: {
    drop(el) {
      for (let i = 0; i < this.balls.length; i++) {
        const ball = this.balls[i]
        if (!ball.show) {
          ball.show = true
          ball.el = el
          this.dropBalls.push(ball)
          return
        }
      }
    },
    beforeDrop(el) {
      const ball = this.dropBalls[this.dropBalls.length - 1]
      const rect = ball.el.getBoundingClientRect()
      const x = rect.left - 32
      const y = -(window.innerHeight - rect.top - 22)
      el.style.display = ''
      el.style.transform = el.style.webkitTransform = `translate3d(0,${y}px,0)`
      const inner = el.getElementsByClassName('inner-hook')[0]
      inner.style.transform = inner.style.webkitTransform = `translate3d(${x}px,0,0)`
    },
    dropping(el, done) {
      this._reflow = document.body.offsetHeight
      el.style.transform = el.style.webkitTransform = `translate3d(0,0,0)`
      const inner = el.getElementsByClassName('inner-hook')[0]
      inner.style.transform = inner.style.webkitTransform = `translate3d(0,0,0)`
      el.addEventListener('transitionend', done)
    },
    afterDrop(el) {
      const ball = this.dropBalls.shift()
      if (ball) {
        ball.show = false
        el.style.display = 'none'
      }
    },
    toggleList() {
      if (this.listFold) {
        if (!this.totalCount) {
          return
        }
        this.listFold = false
        this._showShopCartList()
        this._showShopCartSticky()
      } else {
        this.listFold = true
        this._hideShopCartList()
      }
    },
    pay(e) {
      if (this.totalPrice < this.minPrice) {
        return
      }
      this.dialogComp = this.$createDialog({
        title: '支付',
        content: `支付${this.totalPrice}元`
      })
      this.dialogComp.show()
      e.stopPropagation()
    },
    _showShopCartList() {
      this.shopCartListComp = this.shopCartListComp || this.$createShopCartList({
        $props: {
          selectFoods: 'selectFoods'
        },
        $events: {
          hide: () => {
            this.listFold = true
          },
          leave: () => {
            this._hideShopCartSticky()
          },
          add: (el) => {
            this.shopCartStickyComp.drop(el)
          }
        }
      })
      this.shopCartListComp.show()
    },
    _showShopCartSticky() {
      this.shopCartStickyComp = this.shopCartStickyComp || this.$createShopCartSticky({
        $props: {
          selectFoods: 'selectFoods',
          deliveryPrice: 'deliveryPrice',
          minPrice: 'minPrice',
          fold: 'listFold',
          list: this.shopCartListComp
        }
      })
      this.shopCartStickyComp.show()
    },
    _hideShopCartList() {
      const comp = this.sticky ? this.$parent.list : this.shopCartListComp
      comp.hide && comp.hide()
    },
    _hideShopCartSticky() {
      this.shopCartStickyComp.hide()
    }
  },
  watch: {
    fold(newVal) {
      this.listFold = newVal
    },
    totalCount(newVal) {
      if (!this.listFold && !newVal) {
        this._hideShopCartList()
      }
    }
  },
  components: {
    Bubble
  }
}
</script>

<style lang="stylus" scoped>
  .shopcart
    height: 100%
    .content
      background: #07111b
      color: #999
      display: flex
      font-size: 0
      .content-left
        -webkit-box-flex: 1
        flex: 1
        .logo-wrapper
          background: #07111b
          border-radius: 50%
          box-sizing: border-box
          display: inline-block
          height: 56px
          margin: 0 12px
          padding: 6px
          position: relative
          top: -10px
          vertical-align: top
          width: 56px
          .logo
            background: #333
            border-radius: 50%
            height: 100%
            text-align: center
            width: 100%
            .icon-shopping_cart
              color: #999
              font-size: 24px
              line-height: 44px
            .icon-shopping_cart.highlight
              color: white
              font-size: 24px
              line-height: 44px
          .highlight
            background: #00a0dc
          .num
            position: absolute
            right: 0
            top: 0
        .price
          border-right: 1px solid hsla(0,0%,100%,.1)
          box-sizing: border-box
          display: inline-block
          font-size: 16px
          font-weight: 700
          line-height: 24px
          margin-top: 12px
          padding-right: 12px
          vertical-align: top
          display: inline-block
        .price.highlight
          color: white
        .desc
          display: inline-block
          font-size: 10px
          line-height: 24px
          margin: 12px 0 0 12px
          vertical-align: top
      .content-right
        -webkit-box-flex: 0
        flex: 0 0 105px
        width: 105px
        .pay
          background: #333
          font-size: 12px
          font-weight: 700
          height: 48px
          line-height: 48px
          text-align: center
        .enough
          background: #00b43c
          color: #fff
    .ball-container
      .ball
        position: fixed
        left: 32px
        bottom: 22px
        z-index: 200
        transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
        .inner
          width: 16px
          height: 16px
          border-radius: 50%
          background: $color-blue
          transition: all 0.4s linear
</style>
