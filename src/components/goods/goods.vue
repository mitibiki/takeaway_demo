<template>
    <div class="goods">
        <div class="menu-wrapper" ref="menuWrapper">
            <ul>
                <li v-for="(item, index) in goods" class="menu-item" :class="{'current':currentIndex===index}" @click="selectMenu(index, $event)">
                    <span class="text border-1px">
                        <icon class="icon" v-show="item.type > 0" :type="item.type"></icon>
                        {{item.name}}
                    </span>
                </li>
            </ul>
        </div>
        <div class="foods-wrapper" ref="foodsWrapper">
          <ul>
            <li class="food-list food-list-hook" v-for="item in goods">
              <h1 class="title">{{item.name}}</h1>
              <ul>
                <li class="food-item border-1px" @click="selectFood(food)" v-for="food in item.foods">
                  <div class="icon">
                    <img :src="food.icon" width="57" height="57" alt="商品图标">
                  </div>
                  <div class="content">
                    <h2 class="food-name">{{food.name}}</h2>
                    <p class="desc" v-show="food.description">{{food.description}}</p>
                    <div class="extra">
                      <span>月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                    </div>
                    <div class="price">
                      <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                    </div>
                    <div class="cartcontrol-wrapper">
                      <!-- v-on监听子组件传来的事件 -->
                      <cartcontrol :food="food" v-on:cartAdd="_drop"></cartcontrol>
                    </div>
                  </div>
                </li>
              </ul>
            </li>
          </ul>
        </div>
        <shopcart :selectFoods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" ref="shopCart"></shopcart>
        <food :food="selectedFood" ref="foodInfo" v-on:cartAdd="_drop"></food>
    </div>
</template>

<script>
  import BScroll from 'better-scroll';
  import shopcart from 'components/shopcart/shopcart';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import food from 'components/food/food';
  import icon from 'components/icon/icon';

  const ERR_NO = 0;

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
            scrollY: 0,
            selectedFood: {}
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
      },
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created() {
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_NO) {
            this.goods = response.data;
            this.$nextTick(() => {
              this._initScroll();
              this._calculateHeight();
            });
        }
      });
    },
    methods: {
      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        });
        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          probeType: 3,
          click: true
        });

        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      },
      _calculateHeight() {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;

          // 构造一个递增的高度区间数组
          this.listHeight.push(height);
        }
      },
      selectMenu(index, event) {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el, 300);
      },
      _drop(target) {
        // 优化卡顿，异步执行下落动画
        this.$nextTick(() => {
          // vue2.0 v-el与v-ref合并成ref，统一用$refs来调用
          this.$refs.shopCart.drop(target);
        });
      },
      selectFood(food) {
        this.selectedFood = food;
        // 调用子组件的方法
        this.$refs.foodInfo.show();
      }
    },
    components: {
      shopcart,
      cartcontrol,
      food,
      icon
    }
  };
</script>

<style lang="stylus" rel="sytlesheet/stylus">
  @import '../../common/stylus/mixin.styl'
  .goods
    display: flex
    position: absolute
    top: 174px
    bottom: 46px
    width: 100%
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background-color: #f3f5f7
      .menu-item
          display: table
          height: 54px
          width: 56px
          padding: 0 12px
          line-height: 14px
          &.current
            position: relative
            z-index: 10
            margin-top: -1px
            background-color: white
            .text
              font-weight: 700
              border-none()
          .icon
            display: inline-block
            vertical-align: top
            width: 12px
            height 12px
            background-size: 12px 12px
            background-repeat: no-repeat
          .text
            display: table-cell
            width: 56px
            vertical-align: middle
            font-size: 12px
            border-1px(rgba(7, 17, 27, 0.2))
    .foods-wrapper
      flex: 1
      .title
        height: 26px
        padding-left: 14px
        border-left: 2px solid #d9dde1
        line-height: 26px
        background-color: #f3f5f7
        color: rgb(147, 153, 159)
        font-size: 12px
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, 0.2))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          width: 57px
          margin-right: 10px
        .content
          flex: 1
          .food-name
            margin: 2px 0 8px 0
            line-height: 14px
            font-size: 14px
            color: rgb(7, 17, 27)
          .desc, .extra
            line-height: 10px
            font-size: 10px
            color: rgb(147, 153, 159)
          .desc
            line-height: 12px
            margin-bottom: 8px
          .extra
            span:nth-child(1)
              margin-right: 12px
          .price
            font-weight: 700
            line-height: 24px
            margin-top: 6px
            .now
              margin-right: 8px
              font-size: 14px
              color: rgb(240, 20, 20)
            .old
              text-decoration: line-through
              font-size: 10px
              color: rgb(147, 153, 159)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 12px
</style>
