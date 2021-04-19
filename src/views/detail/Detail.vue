<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll
      class="content"
      ref="scroll"
      :probe-type="3"
      @scroll="contentScroll"
    >
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods"></detail-base-info>
      <detail-shop-info :shop="shop"></detail-shop-info>
      <detail-goods-info
        :detail-info="detailInfo"
        @imageLoad="imageLoad"
      ></detail-goods-info>
      <detail-param-info
        ref="param"
        :param-info="paramInfo"
      ></detail-param-info>
      <detail-comment-info
        ref="comment"
        :comment-info="commentInfo"
      ></detail-comment-info>
      <goods-list ref="recommend" :goods="recommends"></goods-list>
    </scroll>
  </div>
</template>
<script>
import DetailNavBar from "./childComps/DetailNavBar";
import DetailSwiper from "./childComps/DetailSwiper";
import DetailBaseInfo from "./childComps/DetailBaseInfo";
import DetailShopInfo from "./childComps/DetailShopInfo";
import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
import DetailParamInfo from "./childComps/DetailParamInfo";
import DetailCommentInfo from "./childComps/DetailCommentInfo";
import GoodsList from "components/content/goods/GoodsList";

import Scroll from "components/common/scroll/Scroll";

import {
  getDetail,
  Goods,
  Shop,
  GoodsParam,
  getRecommend,
} from "network/detail";
import { debounce } from "common/utils";
import { itemListenerMixin } from "common/mixin";

export default {
  name: "Detail",
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    Scroll,
    GoodsList,
  },
  mixins: [itemListenerMixin],
  data() {
    return {
      iid: null,
      topImages: [],
      goods: {},
      shop: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      recommends: [],
      themTopYs: [0],
      getThemTopY: null,
      currentIndex: 0
    };
  },
  created() {
    // 1:保存传入的iid
    this.iid = this.$route.params.iid;

    // 2：根据iid请求详情数据
    getDetail(this.iid).then((res) => {
      // 1：获取顶部轮播图数据
      // console.log(res);
      const data = res.data.result;
      this.topImages = data.itemInfo.topImages;

      // 2：获取商品数据
      this.goods = new Goods(
        data.itemInfo,
        data.columns,
        data.shopInfo.services
      );

      // 3:获取店铺数据
      this.shop = new Shop(data.shopInfo);

      // 4:保存商品详情数据
      this.detailInfo = data.detailInfo;

      // 5:获取参数信息
      this.paramInfo = new GoodsParam(
        data.itemParams.info,
        data.itemParams.rule
      );

      // 6:取出评论的信息
      if (data.rate.cRate !== 0) {
        this.commentInfo = data.rate.list[0];
      }
    });

    // 3:请求推荐数据
    getRecommend().then((res) => {
      this.recommends = res.data.data.list;
    });

    // 4:getThemTopY赋值  还有防抖操作
    this.getThemTopY = debounce(() => {
      this.$nextTick(() => {
        this.themTopYs = [];
        this.themTopYs.push(0);
        this.themTopYs.push(this.$refs.param.$el.offsetTop - 44);
        this.themTopYs.push(this.$refs.comment.$el.offsetTop - 44);
        this.themTopYs.push(this.$refs.recommend.$el.offsetTop - 44);
        this.themTopYs.push(Number.MAX_VALUE)
        console.log(this.themTopYs);
      }, 200);
    });
  },
  mounted() {},
  updated() {},
  destroyed() {
    this.$bus.$off("itemImageLoad", this.itemImgListener);
  },
  methods: {
    imageLoad() {
      this.$refs.scroll.refresh();
      // 联合
      this.getThemTopY();
    },
    titleClick(index) {
      // console.log(index);
      this.$refs.scroll.scrollTo(0, -this.themTopYs[index], 500);
    },
    contentScroll(position) {
      const positionY = -position.y
      for (let i = 0; i < this.themTopYs.length - 1; i++) {
        if (this.currentIndex !== i && (positionY >= this.themTopYs[i] && positionY <= this.themTopYs[i+1])) {
          this.currentIndex = i;
          this.$refs.nav.currentIndex = this.currentIndex;
        }
      }
    },
  },
};
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
