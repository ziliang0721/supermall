<template>
  <div id="detail">

    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll">
      <detail-swiper :top-images="topImages"/>
      <detail-base-info :goods="goods"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <detail-param-info ref="params" :param-info="paramInfo"/>
      <detail-comment-info ref="comment" :comment-info="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>

    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
    <detail-bottom-bar @addCart="addToCart"/>
  </div>
</template>

<script>
  import  DetailNavBar from './childsComps/DetailNavBar'
  import DetailSwiper from './childsComps/DetailSwiper'
  import DetailBaseInfo from './childsComps/DetailBaseInfo'
  import DetailShopInfo from './childsComps/DetailShopInfo'
  import DetailGoodsInfo from './childsComps/DetailGoodsInfo'
  import DetailParamInfo from './childsComps/DetailParamInfo'
  import DetailCommentInfo from './childsComps/DetailCommentInfo'
  import DetailBottomBar from './childsComps/DetailBottomBar'

  import {getDetail, Goods, Shop, GoodsParam, getRecommend} from "network/detail";
  import {debounce} from "common/utils";
  import {itemListenerMixin, backTopMixin} from "common/mixin";

  import scroll from 'components/common/scroll/Scroll'
  import GoodsList from 'components/content/goods/GoodsList'

  export default {
    name: "Detail",
    data() {
      return{
        iid: null,
        topImages: [],
        goods: {},
        shop: {},
        detailInfo: {},
        paramInfo: {},
        commentInfo: {},
        recommends: [],
        themeTopY: [],
        getThemeTopY: null,
        currentIndex: 0
      }
    },
    created() {
      //1.保存传入的iid
      this.iid = this.$route.params.iid
      //2.根据iid请求详情数据
      getDetail(this.iid).then(res => {
        // console.log(res);
        const data = res.result
        //1.获取顶部的图片轮播数据
        this.topImages = data.itemInfo.topImages

        //2.获取商品信息
        this.goods = new Goods(data.itemInfo, data.columns, data.shopInfo.services)

        //3.创建商店信息
        this.shop = new Shop(data.shopInfo)

        //4.保存商品的详情数据
        this.detailInfo = data.detailInfo

        //5.获取参数的信息
        this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule )

        //6.取出评论的信息
        if (data.rate.cRate !== 0){
          this.commentInfo = data.rate.list[0]
        }
      })

      //3.请求推荐数据
      getRecommend().then(res => {
        // console.log(res);
        this.recommends = res.data.list
      })

      this.getThemeTopY = debounce(() => {
        this.themeTopY = []
        this.themeTopY.push(0)
        this.themeTopY.push(this.$refs.params.$el.offsetTop-44)
        this.themeTopY.push(this.$refs.comment.$el.offsetTop-44)
        this.themeTopY.push(this.$refs.recommend.$el.offsetTop-44)
        this.themeTopY.push(Number.MAX_VALUE)
        // console.log(this.themeTopY);
      }, 100)
    },
    methods: {
      imageLoad() {
        this.$refs.scroll.refresh()
        this.getThemeTopY()

      },
      titleClick(index) {
        this.$refs.scroll.scrollTo(0, -this.themeTopY[index], 200)
      },
      contentScroll(position) {
        //1,获取y值
        const positionY = -position.y
        //2.positionY与主题的Y对比
        let length = this.themeTopY.length
        for (let i = 0; i < length -1; i++){
          if (this.currentIndex !== i && (positionY >=this.themeTopY[i] && positionY
              < this.themeTopY[i+1])) {
            this.currentIndex = i
            this.$refs.nav.currentIndex = this.currentIndex
          }
        }
        //3.是否显示回到顶部
        this.isShowBackTop = (-position.y) > 1000
      },
      addToCart() {
        //1.获取购物车需要展示的信息
        const product = {}
        product.image = this.topImages[0]
        product.title = this.goods.title
        product.desc = this.goods.desc
        product.price = this.goods.realPrice
        product.iid = this.iid

        //2.将商品添加到购物车里
        // this.$store.commit('addCart', product)
        this.$store.dispatch('addCart', product).then(res => {
          this.$toast.show(res, 2000)
        })
      }


    },
    components: {
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      DetailGoodsInfo,
      DetailParamInfo,
      DetailCommentInfo,
      DetailBottomBar,

      scroll,
      GoodsList

    },
    mixins: [itemListenerMixin, backTopMixin],
    mounted() {
    },
   destroyed() {
     this.$bus.$off('itemImgLoad', this.itemImgListener )
   }
  }
</script>

<style scoped>
  #detail {
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
  }
  .detail-nav {
    position: relative;
    z-index: 9;
    background-color: #fff;
  }
  .content {
    height: calc(100% - 44px);
  }
</style>