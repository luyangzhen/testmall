<template>
  <div class="goods-item" @click="itemClick">
    <a>
      <img :src="showImage" alt="" @load="imageLoad"/>
      </a>
      <div class="goods-info">
          <a :href="goodsItem.link"><p>{{goodsItem.title}}</p></a>
          <span class="price">{{goodsItem.price}}</span>
          <span class="cfav">{{goodsItem.cfav}}</span>
      </div>
    
  </div>
</template>

<script>
// console.log(goodslist);
export default {
  name: "GoodsListItem",
  props: {
    // 谁传过来我就以谁命名
    goodsItem: {
      type: Object,
      default() {
        return {};
      },
    },
  },
  computed: {
    showImage() {
      return this.goodsItem.image || this.goodsItem.show.img
    }
  },
  methods: {
    imageLoad() {
      this.$bus.$emit('itemImageLoad')
      // if (this.$route.path.indexOf('./home')) {
      //   this.$bus.$emit('homeItemImageLoad')
      // } else if (this.$route.path.indexOf('./detail')) {
      //   this.$bus.$emit('detailItemImageLoad')
      // }
    },
    itemClick() {
      this.$router.push('/detail' + this.goodsItem.iid)
    
    }
  }
};
</script>

<style scoped>
.goods-item {
    width: 48%;
    position: relative;
    padding-bottom: 40px;
}
.goods-item img {
    width: 100%;
    height: 100%;
    border-radius: 7px;
}
.goods-info {
    position: absolute;
    bottom: 5px;
    left: 0;
    right: 0;
    font-size: 12px;
    text-align: center;
    overflow: hidden;
}
.goods-info p {
    overflow: hidden;
    /* 隐藏的部分用...来代替 */
    text-overflow: ellipsis;
    /* 文本不换行 */
    white-space: nowrap;
    margin-bottom: 3px;
}
.goods-info .price, .goods-info .cfav {
    font-size: 12px;
}
.goods-info .price {
    color: var(--color-high-text);
    margin-right: 20px;
}
.goods-info .cfav {
    position: relative;
}
.goods-info .cfav::before {
    content: '';
    position: absolute; 
    left: -16px;
    top: -1px;
    width: 14px;
    height: 14px;
    background: url("~assets/img/common/collect.svg") 0 0/14px 14px no-repeat;
}
</style>