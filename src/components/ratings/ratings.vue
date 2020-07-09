<template>
  <cube-scroll class="ratings" :data="computedRatings" :options="scrollOptions">
    <div class="ratings-content">
      <div class="overview">
        <div class="overview-left">
          <h1 class="score">{{seller.score}}</h1>
          <div class="title">综合评分</div>
          <div class="rank">高于周边商家{{seller.rankRate}}%</div>
        </div>
        <div class="overview-right">
          <div class="score-wrapper">
            <span class="title">服务态度</span>
            <star :size="36" :score="seller.serviceScore"></star>
            <span class="score">{{seller.serviceScore}}</span>
          </div>
          <div class="score-wrapper">
            <span class="title">商品评分</span>
            <star :size="36" :score="seller.foodScore"></star>
            <span class="score">{{seller.foodScore}}</span>
          </div>
          <div class="delivery-wrapper">
            <span class="title">送达时间</span>
            <span class="delivery">{{seller.deliveryTime}}分钟</span>
          </div>
        </div>
      </div>
      <split></split>
      <rating-select
        :ratings="ratings"
        :onlyContent="onlyContent"
        :selectType="selectType"
        @select="onSelect"
        @toggle="onToggle"
        v-if="ratings.length"
      >
      </rating-select>
      <div class="rating-wrapper">
        <ul>
          <li
            v-for="(rating,index) in computedRatings"
            :key="index"
            class="rating-item border-bottom-1px"
          >
            <div class="avatar">
              <img width="28" height="28" :src="rating.avatar">
            </div>
            <div class="content">
              <h1 class="name">{{rating.username}}</h1>
              <div class="star-wrapper">
                <star :size="24" :score="rating.score"></star>
                <span class="delivery" v-show="rating.deliveryTime">{{rating.deliveryTime}}</span>
              </div>
              <p class="text">{{rating.text}}</p>
              <div class="recommend" v-show="rating.recommend && rating.recommend.length">
                <span class="icon-thumb_up"></span>
                <span
                  class="item"
                  v-for="(item,index) in rating.recommend"
                  :key="index"
                >
                  {{item}}
                </span>
              </div>
              <div class="time">
                {{format(rating.rateTime)}}
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </cube-scroll>
</template>

<script>
import Star from 'components/star/star'
import Split from 'components/split/split'
import RatingSelect from 'components/rating-select/rating-select'
import ratingMixin from 'common/mixins/rating'
import { getRatings } from 'api/index'
import moment from 'moment'

export default {
  name: 'ratings',
  mixins: [ratingMixin],
  props: {
    data: {
      type: Object
    }
  },
  data() {
    return {
      ratings: [],
      scrollOptions: {
        click: false,
        directionLockThreshold: 0
      }
    }
  },
  computed: {
    seller() {
      return this.data.seller || {}
    }
  },
  methods: {
    fetch() {
      if (!this.fetched) {
        this.fetched = true
        getRatings().then((ratings) => {
          this.ratings = ratings
        })
      }
    },
    format(time) {
      return moment(time).format('YYYY-MM-DD hh:mm')
    }
  },
  components: {
    Star,
    Split,
    RatingSelect
  }
}
</script>

<style lang="stylus" scoped>
  .ratings
    height: 100%
    position: relative
    text-align: left
    white-space: normal
    .overview
      display: flex
      padding: 18px 0
      .overview-left
        -webkit-box-flex: 0
        border-right: 1px solid #d9dde1
        flex: 0 0 137px
        padding: 6px 0
        text-align: center
        width: 137px
        .score
          color: #fc9153
          font-size: 24px
          line-height: 28px
          margin-bottom: 6px
        .title
          color: #333
          font-size: 12px
          line-height: 12px
          margin-bottom: 8px
        .rank
          color: #999
          font-size: 10px
          line-height: 10px
      .overview-right
        -webkit-box-flex: 1
        flex: 1
        padding: 6px 0 6px 24px
        .score-wrapper
          -webkit-box-align: center
          align-items: center
          display: -webkit-box
          display: -ms-flexbox
          display: flex
          margin-bottom: 8px
          .title
            color: #333
            font-size: 12px
            line-height: 18px
        .delivery-wrapper
          -ms-flex-align: center
          -webkit-box-align: center
          align-items: center
          display: -webkit-box
          display: -ms-flexbox
          display: flex
          .title
            color: #333
            font-size: 12px
            line-height: 18px
          .delivery
            color: #999
            font-size: 12px
            margin-left: 12px
    .rating-wrapper
      padding: 0 18px
      .rating-item
        display: -webkit-box
        display: -ms-flexbox
        display: flex
        padding: 18px 0
        .avatar
          -ms-flex: 0 0 28px
          -webkit-box-flex: 0
          flex: 0 0 28px
          margin-right: 12px
          width: 28px
          img
            border-radius: 50%
            height: auto
        .content
          -ms-flex: 1
          -webkit-box-flex: 1
          flex: 1
          position: relative
          .name
            color: #333
            font-size: 10px
            line-height: 12px
            margin-bottom: 4px
          .star-wrapper
            -ms-flex-align: center
            -webkit-box-align: center
            align-items: center
            display: -webkit-box
            display: -ms-flexbox
            display: flex
            margin-bottom: 6px
            .star
              margin-right: 6px
              -ms-flex-align: center
              -ms-flex-pack: center
              -webkit-box-align: center
              -webkit-box-pack: center
              align-items: center
              display: -webkit-box
              display: -ms-flexbox
              display: flex
              justify-content: center
              .star-item
                background-size: 10px 10px
                height: 10px
                margin-right: 3px
                width: 10px
            .delivery
              color: #999
              font-size: 10px
          .text
            color: #333
            font-size: 12px
            line-height: 18px
            margin-bottom: 8px
          .recommend
            -ms-flex-align: center
            -ms-flex-wrap: wrap
            -webkit-box-align: center
            align-items: center
            display: -webkit-box
            display: -ms-flexbox
            display: flex
            flex-wrap: wrap
            line-height: 16px
            .icon-thumb_up
              color: #00a0dc
              font-size: 10px
              margin: 0 8px 4px 0
            .item
              background: #fff
              border: 1px solid rgba(7,17,27,.1)
              border-radius: 1px
              color: #999
              padding: 0 6px
              font-size: 10px
              margin: 0 8px 4px 0
          .time
            color: #999
            font-size: 12px
            line-height: 12px
            position: absolute
            right: 0
            top: 0
</style>
