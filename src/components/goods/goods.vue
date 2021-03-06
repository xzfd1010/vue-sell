<template>
  <div>
    <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li v-for="(item,index) in goods" class="menu-item" :class="{'current':currentIndex === index}"
              @click="selectMenu(index,$event)">
          <span class="text border-1px-bottom">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
            {{item.name}}
          </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul>
          <li v-for="item in goods" class="food-list food-list-hook">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li v-for="food in item.foods" class="food-item border-1px-bottom" @click="selectFood(food,$event)">
                <div class="icon">
                  <img width="57" height="57" :src="food.icon" alt="">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">¥{{food.price}}</span>
                    <span v-show="food.oldPrice" class="old">¥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice"
                :min-price="seller.minPrice" ref="shopcart"></shopcart>
    </div>
    <food @add="addFood" :food="selectedFood" ref="food"></food>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'
  import shopcart from 'components/shopcart/shopcart'
  import cartcontrol from 'components/cartcontrol/cartcontrol'
  import food from 'components/food/food'

  const ERR_OK = 0

  export default{
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
      }
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i]
          let height2 = this.listHeight[i + 1]
          // !height2是为了防止最后一个时候，!height2是undefined
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            // 返回当前的索引，对应列表项
            return i
          }
        }
        return 0
      },
      selectFoods() {
        let foods = []
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food)
            }
          })
        })
        return foods
      }
    },
    created() {
//      这里也可以抽象一个组件出来
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
      this.$http.get('/api/goods').then((response) => {
        response = response.body
        if (response.errno === ERR_OK) {
          this.goods = response.data
          // 在下次更新DOM时执行initScroll和calculateHeight，也就是DOM已经存在的时候，应该有别的生命周期函数可以放
          this.$nextTick(() => {
            this._initScroll()
            this._calculateHeight()
          })
        }
      })
    },
    methods: {
      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menuWrapper, {
          click: true
        })

        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true,
//          希望scroll在滚动时，实时告知滚动的位置，应该是个参数吧
          probeType: 3
        })
        this.foodsScroll.on('scroll', (pos) => {
          // 获取滚动的为止，abs
          this.scrollY = Math.abs(Math.round(pos.y))
        })
      },
      _calculateHeight() {
//        找到每个li
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i]
          height += item.clientHeight
//          构造一个递增的区间数组
          this.listHeight.push(height)
        }
      },
      selectMenu(index, event) {
        // 为了防止pc端点击事件派发两次，传递event，自己派发的click和浏览器原生的click有不同，浏览器的没有event._constructed
        if (!event._constructed) {
          return
        }
//        console.log(index)
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
        let el = foodList[index]
        this.foodsScroll.scrollToElement(el, 300)
      },
      _drop(target) {
//        调用shopcart的drop方法，这里利用了异步
//        体验优化，异步执行下落动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target)
        })
      },
      addFood(target) {
        this._drop(target)
      },
      selectFood(food, event) {
        if (!event._constructed) {
          return
        }
        this.selectedFood = food
        this.$refs.food.show()
      }
    },
    components: {
      shopcart,
      cartcontrol,
      food
    }
  }
</script>

<style lang="stylus" type="text/stylus">
  @import "../../common/stylus/mixin.styl"

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
      background: #f3f5f7
      .menu-item
        display: table
        height: 54px
        width: 56px
        padding: 0 12px
        line-height: 14px
        &.current
          z-index: 10
          margin-top: -1px
          line-height: 14px
          background: #fff;
          font-weight: 700
          font-size: 12px
          .text
            border-none()
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat;
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          font-size: 12px
          border-1px-bottom(rgba(7, 17, 27, .1))
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        border-left: 2px solid #d9dde1
        font-size: 12px
        color: rgb(147, 153, 159)
        background: #f3f5f7;
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px;
        border-1px-bottom(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
          margin-bottom: 0;
        .icon
          flex: 0 0 57px;
          margin-right: 10px;
        .content
          flex: 1
          .name
            margin: 2px 0 8px 0;
            height: 14px
            line-height: 14px
            font-size: 14px;
            color: rgb(7, 17, 27)
          .desc, .extra
            line-height: 10px;
            font-size: 10px;
            color: rgb(147, 153, 159)
          .desc
            margin-bottom: 8px;
            line-height: 12px
          .extra
            .count
              margin-right: 12px;
          .price
            font-weight: 700
            line-height: 24px
            .now
              margin-right: 8px
              color: rgb(240, 20, 20)
              font-size: 14px
            .old
              text-decoration: line-through;
              font-size: 10px
              color: rgb(147, 153, 159)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 12px

</style>
