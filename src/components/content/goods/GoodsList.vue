<template>
  <div class="goods">
    <goods-list-item v-for="item in goods" :key="item.id" :goodsItem="item" />
  </div>
</template>

<script>
import GoodsListItem from "./GoodsListItem";
export default {
  props: {
    goods: {
      type: Array,
      default() {
        return [];
      }
    }
  },
  components: {
    GoodsListItem
  }
};
</script>

<style lang="scss" scoped>
.goods {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;

  padding: 2px;
}
</style>
