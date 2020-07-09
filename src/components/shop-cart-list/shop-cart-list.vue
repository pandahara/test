<template>
  <transition name="fade">
    <cube-popup
      v-show="visible"
      :mask-closable=true
      :z-index=90
      position='bottom'
      type="shop-cart-list"
      @mask-click="maskClick"
    >
      <transition
        name="move"
        @after-leave="onLeave"
      >
        <div v-show="visible">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span @click="empty" class="empty">清空</span>
          </div>
          <cube-scroll class="list-content" ref="listContent">
            <ul>
              <li
                class="food"
                v-for="food in selectFoods"
                :key="food.name"
              >
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price*food.count}}</span>
                </div>
                <div class="cart-control-wrapper">
                  <cart-control @add="onAdd" :food="food"></cart-control>
                </div>
              </li>
            </ul>
          </cube-scroll>
        </div>
      </transition>
    </cube-popup>
  </transition>
</template>

<script>
import CartControl from 'components/cart-control/cart-control'

const EVENT_HIDE = 'hide'
const EVENT_LEAVE = 'leave'
const EVENT_ADD = 'add'

export default {
  name: 'shop-cart-list',
  props: {
    selectFoods: {
      type: Array,
      default() {
        return []
      }
    }
  },
  data() {
    return {
      visible: false
    }
  },
  methods: {
    show() {
      this.visible = true
      this.$nextTick(() => {
        this.$refs.listContent.refresh()
      })
    },
    hide() {
      this.visible = false
      this.$emit(EVENT_HIDE)
    },
    onLeave() {
      this.$emit(EVENT_LEAVE)
    },
    maskClick() {
      this.hide()
    },
    onAdd(target) {
      this.$emit(EVENT_ADD, target)
    },
    empty() {
      this.diologComp = this.$createDialog({
        type: 'confirm',
        title: '请空购物车',
        content: '确认清空吗？',
        $events: {
          confirm: () => {
            this.selectFoods.forEach((food) => {
              food.count = 0
            })
            this.hide()
          }
        }
      })
      this.diologComp.show()
    }
  },
  components: {
    CartControl
  }
}
</script>

<style lang="stylus" scoped>
  @import "~common/stylus/variable"
  .cube-shop-cart-list
    bottom: 48px
    &.fade-enter, &.fade-leave-active
      opacity: 0
    &.fade-enter-active, &.fade-leave-active
      transition: all .3s ease-in-out
    .move-enter, .move-leave-active
      transform: translate3d(0, 100%, 0)
    .move-enter-active, .move-leave-active
      transition: all .3s ease-in-out
    .list-header
      height: 40px
      line-height: 40px
      padding: 0 18px
      background: $color-background-ssss
      .title
        float: left
        font-size: $fontsize-medium
        color: $color-dark-grey
      .empty
        float: right
        color: #00a0dc
        font-size: 12px
    .cube-scroll-wrapper
      background: #fff
      max-height: 217px
      overflow: hidden
      padding: 0 18px
      .food
        box-sizing: border-box
        padding: 12px 0
        position: relative
        .name
          color: #333
          font-size: 14px
          line-height: 24px
      .price
        bottom: 12px
        color: #f01414
        font-size: 14px
        font-weight: 700
        line-height: 24px
        position: absolute
        right: 90px
      .cart-control-wrapper
        bottom: 6px
        position: absolute
        right: 0
</style>
