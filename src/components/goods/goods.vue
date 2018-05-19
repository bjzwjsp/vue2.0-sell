<template>
<div>
  <div class="goods">
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li v-for="(item,index) in goods" class="menu-item" :class="{'current':currentIndex === index}" @click="selectMenu(index,$event)">
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px">
              <div class="icon"  >
                <img width="57px" height="57px" :src="food.icon" alt="">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}</span>
                  <span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span>
                  <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @event="getEvent" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>

        </li>
      </ul>
    </div>
    <shopcart ref="shopcart"  :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
  </div>
  <food :food="selectedFood" @add="getFood" ref="food"></food>
</div>
</template>

<script type="text/ecmascript-6">
  import BScroll from "better-scroll";
  import shopcart from "../shopcart/shopcart.vue"
  import cartcontrol from "../cartcontrol/cartcontrol.vue"
  import food from "../food/food.vue"
  const ERR_OK=0;
    export default {
      props: {
        seller: {
          type: Object
        }
      },
      data(){
        return {
          goods:[],
          listHeight:[],
          scrollY:0,
          selectedFood:{}
        }
      },
      computed:{
        selectFoods() {
          //foods会传入购物车组件中，作为购物车的数据来源
          let foods = [];
          this.goods.forEach((good) => {
            good.foods.forEach((food) => {
              //food.count>0表示该商品被选过
              if (food.count) {
                foods.push(food);
              }
            });
          });
          return foods;
        },
        currentIndex() {
          for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >=height1 && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
        }
      },
      created() {
        this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
        //在url里加入参数id可以区分不同商家的数据，返回不同的seller数据
        this.$http.get('/api/goods').then((response) => {
          response = response.body;
          if (response.errno === ERR_OK) {
            this.goods=response.data;
            //console.log(this.goods)
            //this.$nextTick() => 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。
            //数据发生变化后，不能直接更新在dom上，需要在回调函数中刷新DOM,即异步加载DOM
            this.$nextTick(() => {//DOM更新回调函数
              this._initScroll();
              this._calculateHeight();
            });
          }
        });
      },
      methods:{
        getFood(el) {
          this.$nextTick(() => {
            this.$refs.shopcart.drop(el);
          });
        },
        selectMenu(index,event){
          if(!event._constructed){
            return;
          };
          let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
          let el = foodList[index]; // 插件的方法，将滚动区域内滚动到某个元素位置，经历300s
          this.foodsScroll.scrollToElement(el, 300);
          // console.log(index)
        },
        _initScroll(){
          //this.$refs：取得DOM对象
          // click: true, 取消默认阻止事件
         // probeType: 3   监听事件的触发时间，1为即时触发，3为延迟到事件完毕后触发
          this.menuScrool  = new BScroll(this.$refs.menuWrapper,{click:true});
          this.foodsScroll = new BScroll(this.$refs.foodsWrapper,{click:true,probeType:3});//得到滚动对象
          //监听滚动事件
          this.foodsScroll.on("scroll",(pos) =>{
              this.scrollY=Math.abs(Math.round(pos.y));
          });

        },
        _calculateHeight(){
          let foodList=this.$refs.foodsWrapper.getElementsByClassName("food-list-hook");
          let height=0;
          this.listHeight.push(height);
          for (let i = 0; i < foodList.length; i++) {
            let item = foodList[i];
            height += item.clientHeight;
            this.listHeight.push(height);
          }
        },
        selectFood(food, event) {
          if (!event._constructed) {
            return;
          }
          this.selectedFood = food;
          this.$refs.food.show();
        },
        getEvent(el) {
          // 体验优化,异步执行下落动画
          this.$nextTick(() => {
            this.$refs.shopcart.drop(el);
          });
        }
      },
      components:{
        shopcart,
        cartcontrol,
        food
      }
    }
</script>
<style lang="stylus" rel="stylesheet/stylus">
  @import '../../common/stylus/mixin.styl';
  .goods
    position: absolute
    top: 174px
    bottom: 46px
    width: 100%
    display: flex
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
          position: relative
          z-index: 10
          margin-top: -1px
          background: #fff
          font-weight: 700
          .text
            border-none()
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
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
          border-1px(rgba(7, 17, 27, 0.1))
          font-size: 12px
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height:26px
        line-height: 26px
        font-size: 12px
        border-left: 2px solid #d9dde1
        color: rgb(147,153,159)
        background: #f3f5f7
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, 0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          margin-right: 10px
        .content
          flex: 1
          .name
            margin: 2px 0 8px 0
            height: 14px
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
            .count
              margin-right: 12px
          .price
            font-weight: 700
            line-height: 24px
            vertical-align: middle
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
