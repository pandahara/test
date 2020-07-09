<template>
  <cube-scroll class="seller" :options="sellerScrollOptions">
    <div class="seller-content">
      <div class="overview">
        <h1 class="title">{{seller.name}}</h1>
        <div class="desc border-bottom-1px">
          <star :size="36" :score="seller.score"></star>
          <span class="text">({{seller.ratingCount}})</span>
          <span class="text">月售{{seller.sellCount}}单</span>
        </div>
        <ul class="remark">
          <li class="block">
            <h2>起送价</h2>
            <div class="content">
              <span class="stress">{{seller.minPrice}}</span>元
            </div>
          </li>
          <li class="block">
            <h2>商家配送</h2>
            <div class="content">
              <span class="stress">{{seller.deliveryPrice}}</span>元
            </div>
          </li>
          <li class="block">
            <h2>平均配送时间</h2>
            <div class="content">
              <span class="stress">{{seller.deliveryTime}}</span>分钟
            </div>
          </li>
        </ul>
        <div class="favorite">
          <span class="icon-favorite" :class="{'active':favorite}"></span>
          <span class="text">{{favoriteText}}</span>
        </div>
      </div>
      <split></split>
      <div class="bulletin">
        <h1 class="title">公告与活动</h1>
        <div class="content-wrapper border-bottom-1px">
          <p class="content">{{seller.bulletin}}</p>
        </div>
        <ul v-if="seller.supports" class="supports">
          <li
            class="support-item border-bottom-1px"
            v-for="(item,index) in seller.supports"
            :key="index"
          >
            <support-ico :size=4 :type="seller.supports[index].type"></support-ico>
            <span class="text">{{seller.supports[index].description}}</span>
          </li>
        </ul>
      </div>
      <split></split>
      <div class="pics">
        <h1 class="title">商家实景</h1>
        <cube-scroll class="pic-wrapper" :options="picScrollOptions">
          <ul class="pic-list">
            <li
              class="pic-item"
              v-for="(pic,index) in seller.pics"
              :key="index"
            >
              <img :src="pic" width="120" height="90">
            </li>
          </ul>
        </cube-scroll>
      </div>
      <split></split>
      <div class="info">
        <h1 class="title border-bottom-1px">商家信息</h1>
        <ul>
          <li
            class="info-item border-bottom-1px"
            v-for="(info,index) in seller.infos"
            :key="index"
          >
            {{info}}
          </li>
        </ul>
      </div>
    </div>
  </cube-scroll>
</template>

<script>
import Star from 'components/star/star'
import Split from 'components/split/split'
import SupportIco from 'components/support-ico/support-ico'

export default {
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
      favorite: false,
      sellerScrollOptions: {
        directionLockThreshold: 0,
        click: false
      },
      picScrollOptions: {
        scrollX: true,
        stopPropogation: true,
        directionLockThreshold: 0
      }
    }
  },
  computed: {
    seller() {
      return this.data.seller || {}
    },
    favoriteText() {
      return this.favorite ? '已收藏' : '收藏'
    }
  },
  components: {
    SupportIco,
    Star,
    Split
  }
}
</script>

<style lang="stylus" scoped>
.seller
  height: 100%
  text-align: left
  .overview
    padding: 18px
    position: relative
    .title
      color: #333
      font-size: 14px
      line-height: 14px
      margin-bottom: 8px
    .desc
      -ms-flex-align: center
      -webkit-box-align: center
      align-items: center
      display: -webkit-box
      display: -ms-flexbox
      display: flex
      padding-bottom: 18px
      .star
        margin-right: 8px
        -ms-flex-align: center
        -ms-flex-pack: center
        -webkit-box-align: center
        -webkit-box-pack: center
        align-items: center
        display: -webkit-box
        display: -ms-flexbox
        display: flex
        justify-content: center
      .text
        color: #666
        font-size: 10px
        line-height: 18px
        margin-right: 12px
    .remark
      display: -webkit-box
      display: -ms-flexbox
      display: flex
      padding-top: 18px
      .block
        -ms-flex: 1
        -webkit-box-flex: 1
        border-right: 1px solid #d9dde1
        flex: 1
        text-align: center
        h2
          color: #999
          font-size: 10px
          line-height: 10px
          margin-bottom: 4px
        .content
          color: #333
          font-size: 10px
          line-height: 24px
          .stress
            font-size: 24px
    .favorite
      position: absolute
      right: 11px
      text-align: center
      top: 18px
      width: 50px
      .icon-favorite
        color: #ccc
        display: block
        font-size: 24px
        line-height: 24px
        margin-bottom: 4px
      .icon-favorite.active
          color: #f01414
      .text
        color: #666
        font-size: 10px
        line-height: 10px
  .bulletin
    padding: 18px 18px 0
    white-space: normal
    .title
      color: #333
      font-size: 14px
      line-height: 14px
      margin-bottom: 8px
    .content-wrapper
      padding: 0 12px 16px
      .content
        color: #f01414
        font-size: 12px
        line-height: 24px
    .supports
      .support-item
        -ms-flex-align: center
        -webkit-box-align: center
        align-items: center
        display: -webkit-box
        display: -ms-flexbox
        display: flex
        padding: 16px 12px
        .support-ico
          margin-right: 6px
        .text
          color: #333
          font-size: 12px
          line-height: 16px
  .pics
    padding: 18px
    .title
      color: #333
      font-size: 14px
      line-height: 14px
      margin-bottom: 12px
    .pic-wrapper
      -ms-flex-align: center
      -webkit-box-align: center
      align-items: center
      display: -webkit-box
      display: -ms-flexbox
      display: flex
      .pic-list
        .pic-item
          display: inline-block
          height: 90px
          margin-right: 6px
          width: 120px
  .info
    color: #333
    padding: 18px 18px 0
    .title
      font-size: 14px
      line-height: 14px
      padding-bottom: 12px
    ul
      .info-item
        font-size: 12px
        line-height: 16px
        padding: 16px 12px
</style>
