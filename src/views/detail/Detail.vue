<template>
  <div id="detail">
    <detail-nav-bar class="detail-navbar" @titleClick="titleClick" ref="nav"></detail-nav-bar>
    <scroll class="scroll-content" ref="scroll" @scroll="contentScroll" :probe-type="3">
      <detail-swiper :top-images="topImages"></detail-swiper>
      <detail-base-info :goods="goods" />
      <detail-shop-info :shop="shop" />
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad" />
      <detail-param-info :param-info="paramInfo" ref="param" />
      <detail-comment-info :comment-info="commentInfo" ref="comment" />
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop" />
    <detail-Bottom-bar @addToCart="addToCart" />
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
import DetailBottomBar from "./childComps/DetailBottomBar";

import {
  getDetail,
  Goods,
  Shop,
  GoodsParam,
  getRecommend,
} from "network/detail";

import Scroll from "components/common/scroll/Scroll";
import BackTop from "components/content/backTop/BackTop";

export default {
  name: "Detail",
  data() {
    return {
      iid: null,
      topImages: [],
      goods: {},
      shop: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      themeTopYs: [],
      currentIndex: 0,
      isShowBackTop: false,
    };
  },
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    Scroll,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    DetailBottomBar,
    BackTop,
  },
  created() {
    this.iid = this.$route.params.iid;

    getDetail(this.iid).then((res) => {
      const data = res.result;

      this.topImages = data.itemInfo.topImages;

      this.goods = new Goods(
        data.itemInfo,
        data.columns,
        data.shopInfo.services
      );

      this.shop = new Shop(data.shopInfo);

      this.detailInfo = data.detailInfo;

      this.paramInfo = new GoodsParam(
        data.itemParams.info,
        data.itemParams.rule
      );

      if (data.rate.list !== 0) {
        this.commentInfo = data.rate.list[0];
      }
    });
  },
  updated() {
    this.themeTopYs = [];

    this.themeTopYs.push(0);
    this.themeTopYs.push(this.$refs.param.$el.offsetTop - 44);
    this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
  },
  methods: {
    imageLoad() {
      this.$refs.scroll.refresh();
    },
    titleClick(index) {
      this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200);
    },
    contentScroll(position) {
      const positionY = -position.y;
      let length = this.themeTopYs.length;
      this.isShowBackTop = -position.y > 1000;
      for (let i = 0; i < length; i++) {
        let iPos = this.themeTopYs[i];
        if (positionY > iPos && positionY < this.themeTopYs[i + 1]) {
          if (this.currentIndex !== i) {
            this.currentIndex = i;
            this.$refs.nav.currentIndex = this.currentIndex;
          }
          break;
        }
      }
    },
    backClick() {
      this.$refs.scroll.scrollTo(0, 0);
    },
    addToCart() {
      const product = {};
      product.image = this.topImages[0];
      product.title = this.goods.title;
      product.desc = this.goods.desc;
      product.price = this.price;
      product.iid = this.iid;

      this.$store.commit("addToCart", product);
    },
  },
};
</script>

<style>
#detail {
  position: relative;
  z-index: 9;
  background-color: #fff;
}
.detail-navbar {
  position: relative;
  z-index: 9;
  background-color: #fff;
}
.scroll-content {
  height: calc(100vh - 44px);
}
</style>
