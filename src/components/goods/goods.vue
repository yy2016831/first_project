<template>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li v-for="(item, index) in goods" :key="index" class="menu-item" :class="{'current':currentIndex === index}"
            @click="selectMenu(index, $event)">
          <span class="text border-1px">
            <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>
            {{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="(item, index) in goods" :key="index" class="food-list foods-list-hook" ref="foodList">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li v-for="(food, index) in item.foods" class="food-item border-1px" :key="index">
              <div class="icon">
                <img :src="food.icon" alt="">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span>
                  <span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span>
                  <span class="old" v-show="food.oldprice">￥{{food.oldPrice}}</span>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import shopcart from '../shopcart/shopcart';

  const ERR_OK = 0;

  export default {
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0
      };
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
      }
    },
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];

      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
    },
    methods: {
      selectMenu(index, event) {
        if (!event._constructed) {
          return;
        }
        let foodList = this.$refs.foodList;
        let el = foodList[index];
        this.foodScroll.scrollToElement(el, 300);
      },
      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        })
        ;
        this.foodScroll = new BScroll(this.$refs.foodsWrapper, {
          probeType: 3
        });
        this.foodScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      },
      _calculateHeight() {
        let foodList = this.$refs.foodList;
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      }
    },
    components: {
      shopcart
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl";

  .goods
    display: flex;
    position: absolute;
    top: 172px;
    bottom: 46px;
    width: 100%;
    overflow: hidden;
    .menu-wrapper
      flex: 0 0 80px;
      width: 80px;
      background: #f3f5f7;
      .menu-item
        display: table;
        padding: 0 12px;
        width: 56px;
        height: 54px;
        line-height: 14px;
        &.current
          position: relative;
          z-index: 10;
          margin-top: -1px;
          background: #fff;
          font-weight: 700;
          .text
            border-none();
        .icon
          display: inline-block;
          width: 12px;
          height 12px;
          margin-right: 2px;
          vertical-align: top;
          background-size: 12px 12px;
          background-repeat: no-repeat;
          &.decrease
            bg-img('decrease_3');
          &.discount
            bg-img('discount_3');
          &.guarantee
            bg-img('guarantee_3');
          &.invoice
            bg-img('invoice_3');
          &.special
            bg-img('special_3');
        .text
          display: table-cell;
          width: 56px;
          vertical-align: middle;
          border-1px(rgba(7, 17, 27, .1));
          font-size: 12px;
    .foods-wrapper
      flex: 1;
      .title
        padding-left: 14px;
        height: 26px;
        line-height: 26px;
        border-left: 2px solid #d9dde1;
        font-size: 12px;
        color: rgb(147, 153, 159);
        background: #f3f5f7;
      .food-item
        display: flex;
        margin: 18px;
        padding-bottom: 18px;
        border-1px(rgba(7, 17, 27, .1));
        &:last-child
          border-none();
          margin-bottom: 0;
        .icon
          flex: 0 0 57px;
          margin-right: 10px;
          & > img
            width: 57px;
            heigth: 57px;
        .content
          flex: 1;
          .name
            margin: 2px 0 8px 0;
            height: 14px;
            line-height: 14px;
            font-size: 14px;
            color: rgb(7, 17, 27);
          .desc, .extra
            line-height: 10px;
            font-size: 10px;
            color: rgb(147, 153, 159);
          .desc
            margin-bottom: 8px;
            line-height: 14px;
          .extra, .price
            font-size: 0;
          .extra
            & > span
              font-size 10px;
            .count
              margin-right: 12px;
          .price
            font-weight: 700;
            line-height: 24px;
            .now
              margin-right: 8px;
              font-size: 14px;
              color: rgb(240, 20, 20);
            .old
              text-decoration: line-through;
              font-size: 10px;
              color: rgb(147, 153, 159);
</style>
