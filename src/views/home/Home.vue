<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control
      :titles="['流行', '新款', '精选']"
      @tabControlClick="tabControlClick"
      ref="tabControl1"
      class="tabControl"
      v-show="isTabFixed"
    />

    <div class="wrapper" ref="wrapper">
      <div class="content">
        <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad" />
        <recommend-view :recommends="recommends" />
        <feature-view />
        <tab-control
          :titles="['流行', '新款', '精选']"
          @tabControlClick="tabControlClick"
          ref="tabControl2"
        />
        <goods-list :goods="showGoods" />
      </div>
    </div>
    <back-top @click.native="backClick" v-show="isShowBackTop" />

    <!-- <scroll class="content" ref="scroll"></scroll> -->
  </div>
</template>

<script>
import HomeSwiper from "./childCompos/HomeSwiper";
import RecommendView from "./childCompos/RecommendView";
import FeatureView from "./childCompos/FeatureView";
import GoodsList from "components/content/goods/GoodsList";

import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";
// import Scroll from "components/common/scroll/Scroll";
import BScroll from "better-scroll";
import BackTop from "components/content/backTop/BackTop";

import { getHomeMultidata, getHomeGoods } from "network/home";

export default {
  name: "Home",
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] }
      },
      currentType: "pop",
      scroll: null,
      isShowBackTop: false,
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0
    };
  },
  components: {
    HomeSwiper,
    RecommendView,
    FeatureView,
    GoodsList,
    NavBar,
    TabControl,
    // Scroll
    BackTop
  },
  created() {
    // 1.请求多个数据
    this.getHomeMultidata();

    // 2.请求商品数据
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  mounted() {
    this.scroll = new BScroll(this.$refs.wrapper, {
      click: true,
      probeType: 3,
      pullUpLoad: true
    });
    this.scroll.on("scroll", position => {
      // 显示一键到顶按钮
      this.isShowBackTop = -position.y > 1000;

      // 吸顶
      this.isTabFixed = -position.y > this.tabOffsetTop;
    });
    // 上拉加载
    this.scroll.on("pullingUp", () => {
      this.getHomeGoods(this.currentType);
    });

    // const refresh = this.debounce(this.scroll.refresh, 500);
    // 3.监听item中图片加载完成
    this.$bus.$on("itemImageLoad", () => {
      this.scroll.refresh();
      // refresh();
    });
  },
  destroyed() {},
  activated() {
    this.scroll.scrollTo(0, this.saveY, 0);
    this.scroll.refresh();
  },
  deactivated() {
    this.saveY = this.scroll.y;
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list;
    }
  },
  methods: {
    // 回顶部
    backClick() {
      // scrollTo(x, y, time)
      this.scroll && this.scroll.scrollTo(0, 0, 300);
    },
    // 防抖
    debounce(func, delay) {
      let timer = null;
      return function(...args) {
        if (timer) clearTimeout(timer);
        timer = setTimeout(() => {
          func.apply(this, args);
        }, delay);
      };
    },
    /**
     * 事件监听相关的方法
     */
    tabControlClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;

        case 1:
          this.currentType = "new";
          break;

        case 2:
          this.currentType = "sell";
          break;
      }
      this.$refs.tabControl1.currentIndex = index;
      this.$refs.tabControl2.currentIndex = index;
    },

    /**
     * 网络请求相关的方法
     */
    getHomeMultidata() {
      getHomeMultidata().then(res => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },
    getHomeGoods(type) {
      const page = this.goods[type].page + 1;
      getHomeGoods(type, page).then(res => {
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;

        this.scroll.finishPullUp();
      });
    },
    swiperImageLoad() {
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop;
    }
  }
};
</script>

<style lang="scss">
#home {
  height: 100vh;
  position: relative;
}

.home-nav {
  background-color: var(--color-tint);
  color: #fff;

  // position: fixed;
  // top: 0;
  // left: 0;
  // right: 0;
  // z-index: 9;
}

// .tab-control {
//   position: sticky;
//   top: 43.5px;
//   z-index: 9;
// }

.tabControl {
  position: relative;
  z-index: 9;
}

.wrapper {
  overflow: hidden;
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
}
</style>
