<template>
  <transition
    name="move"
    @after-leave="afterLeave"
  >
    <div class="food" v-show="visible">
      <cube-scroll ref="scroll">
        <div class="food-content">
          <div class="image-header">
            <img :src="food.image">
            <div class="back" @click="hide">
              <i class="icon-arrow_lift"></i>
            </div>
          </div>
          <div class="content">
            <h1 class="title">{{food.name}}</h1>
            <div class="detail">
              <span class="sell-count">月售{{food.sellCount}}份</span>
              <span class="rating">好评率{{food.rating}}</span>
            </div>
            <div class="price">
              <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
            </div>
            <div class="cart-control-wrapper">
              <cart-control @add="addFood" :food="food"></cart-control>
            </div>
            <transition name="fade">
              <div @click.stop="addFirst" class="buy" v-show="!food.count">
                加入购物车
              </div>
            </transition>
          </div>
          <split v-show="food.info"></split>
          <div class="info" v-show="food.info">
            <h1 class="title">商品信息</h1>
            <p class="text">{{food.info}}</p>
          </div>
          <split></split>
          <div class="rating">
            <h1 class="title">商品评价</h1>
            <rating-select
              :ratings="ratings"
              :onlyContent="onlyContent"
              :selectType="selectType"
              :desc="desc"
              @select="onSelect"
              @toggle="onToggle"
            >

            </rating-select>
            <div class="rating-wrapper">
              <ul v-show="computedRatings && computedRatings.length">
                <li
                  v-for="(rating,index) in computedRatings"
                  :key="index"
                  class="rating-item border-bottom-1px"
                >
                  <div class="user">
                    <span class="name">{{rating.username}}</span>
                    <img class="avatar" width="12" height="12" :src="rating.avatar">
                  </div>
                  <div class="time">{{format(rating.rateTime)}}</div>
                  <p class="text">
                    <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
                  </p>
                </li>
              </ul>
              <div class="no-rating" v-show="!computedRatings || !computedRatings.length">暂无评价</div>
            </div>
          </div>
        </div>
      </cube-scroll>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
import popupMixin from 'common/mixins/popup'
import Split from 'components/split/split'
import CartControl from 'components/cart-control/cart-control'
import RatingSelect from 'components/rating-select/rating-select'
import moment from 'moment'

const EVENT_SHOW = 'show'
const EVENT_LEAVE = 'leave'
const EVENT_ADD = 'add'

const ALL = 2

export default {
  mixins: [popupMixin],
  name: 'food',
  props: {
    food: {
      type: Object
    }
  },
  data() {
    return {
      onlyContent: true,
      selectType: ALL,
      desc: {
        all: '全部',
        positive: '推荐',
        negative: '吐槽'
      }
    }
  },
  computed: {
    ratings() {
      return this.food.ratings
    },
    computedRatings() {
      const ret = []
      this.ratings.forEach((rating) => {
        if (this.onlyContent && !rating.text) {
          return
        }
        if (this.selectType === ALL || this.selectType === rating.rateType) {
          ret.push(rating)
        }
      })
      return ret
    }
  },
  created() {
    this.$on(EVENT_SHOW, () => {
      this.$nextTick(() => {
        this.$refs.scroll.refresh()
      })
    })
  },
  methods: {
    afterLeave() {
      this.$emit(EVENT_LEAVE)
    },
    addFirst(event) {
      this.$set(this.food, 'count', 1)
      this.$emit(EVENT_ADD, event.target)
    },
    addFood(target) {
      this.$emit(EVENT_ADD, target)
    },
    format(time) {
      return moment(time).format('YYYY-MM-DD hh:mm')
    },
    onSelect(type) {
      this.selectType = type
    },
    onToggle() {
      this.onlyContent = !this.onlyContent
    }
  },
  components: {
    Split,
    CartControl,
    RatingSelect
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "~common/stylus/variable"
@import "~common/stylus/mixin.styl"

.food
  position: fixed
  left: 0
  top: 0
  bottom: 48px
  z-index: 30
  width: 100%
  background: $color-white
  &.move-enter-active, &.move-leave-active
    transition: all 0.3s linear
  &.move-enter, &.move-leave-active
    transform: translate3d(100%, 0, 0)
  .image-header
    position: relative
    width: 100%
    height: 0
    padding-top: 100%
    img
      position: absolute
      top: 0
      left: 0
      width: 100%
      height: 100%
    .back
      position: absolute
      top: 10px
      left: 0
      .icon-arrow_lift
        display: block
        padding: 10px
        font-size: $fontsize-large-xx
        color: $color-white
  .content
    position: relative
    padding 18px
    .title
      color: #333
      font-size: 14px
      font-weight: 700
      line-height: 14px
      margin-bottom: 8px
    .detail
      height: 10px
      line-height: 10px
      margin-bottom: 18px
      .sell-count
        margin-right: 12px
        color: #999
        font-size: 10px
      .rating
        color: #999
        font-size: 10px
        padding-top: 18px
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
      right: 12px
    .buy
      background: #00a0dc
      border-radius: 12px
      bottom: 18px
      box-sizing: border-box
      color: #fff
      font-size: 10px
      height: 24px
      line-height: 24px
      opacity: 1
      padding: 0 12px
      position: absolute
      right: 18px
      z-index: 10
  .info
    padding: 18px
    .title
      color: #333
      font-size: 14px
      line-height: 14px
      margin-bottom: 6px
    .text
      color: #666
      font-size: 12px
      line-height: 24px
      padding: 0 8px
  .rating
    padding-top: 18px
    .title
      color: #333
      font-size: 14px
      line-height: 14px
      margin-left: 18px
    .rating-select
      .rating-type
        margin: 0 18px
        padding: 18px 0
    .rating-wrapper
      padding: 0 18px
      .rating-item
        padding: 16px 0
        position: relative
        list-style: none
        .user
          -webkit-box-align: center
          align-items: center
          display: -webkit-box
          display: -ms-flexbox
          display: flex
          line-height: 12px
          position: absolute
          right: 0
          top: 16px
          .name
            color: #999
            font-size: 10px
            margin-right: 6px
          .avatar
            border-radius: 50%
        .time
          color: #999
          font-size: 10px
          line-height: 12px
          margin-bottom: 6px
        .text
          color: #333
          font-size: 12px
          line-height: 16px
          .icon-thumb_up
            color: #00a0dc
          .icon-thumb_down
            color: #999
      .no-rating
        color: #999
        font-size: 12px
        padding: 16px 0
</style>
