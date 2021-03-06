<template>
  <transition name="move">
    <div class="food" v-show="showFlag" ref="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image" alt="foodImage">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="now">¥{{food.price}}</span><span class="old" v-show="food.oldPrice">¥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol @add="addFood" :food="food"></cartcontrol>
          </div>
          <transition name="fade">
            <div class="buy" v-show="!food.count || food.count === 0" @click.stop.prevent="addFirst">
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
          <ratingselect @select="selectRating" @toggle="toggleContent" :ratings="food.ratings" :select-type="selectType" :only-content="onlyContent"
                        :desc="desc"></ratingselect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li class="rating-item" v-show="needShow(rating.rateType, rating.text)" v-for="rating in food.ratings">
                <div class="time">{{rating.rateTime | formatDate}}</div>
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img class="avatar" width="12" height="12" :src="rating.avatar" alt="rating.avatar">
                </div>
                <p class="text">
                <span
                  :class="{'icon-thumb_up':rating.rateType === 0,'icon-thumb_down':rating.rateType === 1}"></span>{{rating.text}}
                </p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import Vue from 'vue';
  import { formatDate } from '../../common/js/date.js';
  import Cartcontrol from 'components/cartcontrol/cartcontrol.vue';
  import Split from 'components/split/split.vue';
  import Ratingselect from 'components/ratingselect/ratingselect.vue';
  import BScroll from 'better-scroll';

  const ALL = 2;

  export default {
    props: {
      food: {
        type: Object
      }
    },
    data() {
      return {
        showFlag: false,
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',
          positive: '推荐',
          negative: '吐槽'
        }
      };
    },
    methods: {
      show() {
        this.showFlag = true;
        this.selectType = ALL;
        this.onlyContent = true;
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$refs.food, {
              click: true
            });
          } else {
            this.scroll.refresh();
          }
        });
      },
      hide() {
        this.showFlag = false;
      },
      selectRating(type) {
          this.selectType = type;
          this.$nextTick(() => {
            this.scroll.refresh();
          });
        },
      toggleContent() {
        this.onlyContent = !this.onlyContent;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },
      addFirst(event) {
        if (!event._constructed) {
          return;
        }
        if (!this.food.count) {
          Vue.set(this.food, 'count', 1);
        } else {
          this.food.count++;
        }
        this.$emit('add', event.target);
      },
      addFood(target) {
        this.$emit('add', target);
      },
      needShow(type, text) {
        if (this.onlyContent && !text) {
          return false;
        }
        if (this.selectType === ALL) {
          return true;
        } else {
          return type === this.selectType;
        }
      }
    },
    filters: {
      formatDate(time) {
        let date = new Date(time);
        return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },
    components: {
      Cartcontrol,
      Split,
      Ratingselect
    }
  };
</script>

<style lang="stylus" rel="stylesheet">
  @import "~common/stylus/mixin.styl"

  .food
    position: fixed
    left: 0
    top: 0
    bottom: 48px
    z-index: 30
    width: 100%
    background-color: #fff
    &.move-enter-active, &.move-leave-active
      transition: all .2s linear
      transform: translate3d(0, 0, 0)
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
          font-size: 20px
          color: #fff
    .content
      position: relative
      padding: 18px
      .title
        margin-bottom: 8px
        line-height: 14px
        font-size: 14px
        font-weight: 700
        color: #07111b
      .detail
        margin-bottom: 18px
        font-size: 10px
        line-height: 10px
        color: #c3c6c9
        .count
          margin-right: 12px
      .price
        margin-bottom: 18px
        font-weight: 700
        .now
          margin-right: 8px
          line-height: 24px
          font-size: 14px
          color: #f01414
        .old
          text-decoration: line-through
          line-height: 20px
          font-size: 10px
          color: #c3c6c9
      .cartcontrol-wrapper
        position: absolute
        right: 12px
        bottom: 12px
      .buy
        position: absolute
        right: 18px
        bottom: 18px
        z-index: 10
        height: 24px
        line-height: 24px
        padding: 0 12px
        box-sizing: border-box
        border-radius: 12px
        background-color: #00a0dc
        font-size: 10px
        color: #fff
        opacity: 1
        &.fade-enter-active, &.fade-leave-active
          transition: all .5s
        &.fade-enter, &.fade-leave-active
          opacity: 0
    .info
      padding: 18px
      .title
        margin-bottom: 6px
        line-height: 14px
        font-size: 14px
        font-weight: 700
        color: #07111b
      .text
        padding: 0 8px
        line-height: 24px
        font-size: 12px
        color: #a3a6a9
    .rating
      padding: 18px
      .title
        line-height: 14px
        font-size: 14px
        font-weight: 700
        color: #07111b
      .rating-item
        position: relative
        padding: 16px 0
        border-1px(rgba(7, 17, 27, .1))
        .user
          position: absolute
          right: 0
          top: 16px
          line-height: 12px
          font-size: 0
          .name
            display: inline-block
            margin-right: 6px
            vertical-align: top
            font-size: 10px
            color: rgb(147, 153, 159)
          .avatar
            border-radius: 50%
        .time
          margin-bottom: 6px
          line-height: 12px
          font-size: 10px
          color: rgb(147, 153, 159)
        .text
          line-height: 16px
          font-size: 12px
          color: rgb(7, 17, 27)
          .icon-thumb_up, .icon-thumb_down
            margin-right: 4px
            line-height: 16px
            font-size: 12px
          .icon-thumb_up
            color: rgb(0, 160, 220)
          .icon-thumb_down
            color: rgb(147, 153, 159)
      .no-rating
        padding: 16px 0
        font-size: 12px
        color: rgb(147, 153, 159)
</style>
