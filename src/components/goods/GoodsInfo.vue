<template>
  <div class="goodsinfo-container">
    <transition
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter">
      <div class="ball" v-show="ballFlag" ref="ball"></div>
    </transition>

    <!-- 商品轮播图 -->
    <div class="mui-card">
      <div class="mui-card-content">
        <div class="mui-card-content-inner">
          <swiper :lunbotuList="lunbotu" :isfull="false"></swiper>
        </div>
      </div>
    </div>

    <!-- 商品购买区域 -->
    <div class="mui-card">
      <div class="mui-card-header">{{ goodsinfo.title }}</div>
      <div class="mui-card-content">
        <div class="mui-card-content-inner">
          <p class="price">
            市场价：
            <del>¥{{ goodsinfo.market_price }}</del>&nbsp;&nbsp;销售价
            <span class="now_price">¥{{ goodsinfo.sell_price }}</span>
          </p>
          <p>
            购买数量：
            <numbox @getcount="getSelectedCount" :max="goodsinfo.stock_quantity"></numbox>
          </p>
          <p>
            <mt-button type="primary" size="small">立即购买</mt-button>
            <mt-button type="danger" size="small" @click="addToShopCar">加入购物车</mt-button>

            <!-- 分析：如何实现加入购物车的时候，拿到 选择器的数量 -->
            <!-- 1. 经过分析发现：按钮属于 goodsinfo 页面，数字属于 numerbox 组件 -->
            <!-- 2. 由于涉及到了父子组件的嵌套了，所以，无法直接再 goodsinfo 页面中 获取到 选中的商品数量 -->
            <!-- 3. 子组件向父组件传值了（时间调用机制） -->
            <!-- 4. 事件调用的本质： 父向子传递方法，子调用这个方法，同时把 数据当作参数 传递给这个方法 -->
          </p>
        </div>
      </div>
    </div>

    <!-- 商品参数区域 -->
    <div class="mui-card">
      <div class="mui-card-header">商品参数</div>
      <div class="mui-card-content">
        <div class="mui-card-content-inner">
          <p>商品货号：{{ goodsinfo.goods_no }}</p>
          <p>库存情况：{{ goodsinfo.stock_quantity }}件</p>
          <p>上架时间：{{ goodsinfo.add_time | dateFormat }}</p>
        </div>
      </div>
      <div class="mui-card-footer">
        <mt-button type="primary" size="large" plain @click="goDesc(id)">图文介绍</mt-button>
        <mt-button type="danger" size="large" plain @click="goComment(id)">商品评论</mt-button>
      </div>
    </div>
  </div>
</template>

<script>
import swiper from "../subcomponnets/swiper.vue";
import numbox from "../subcomponnets/goodsinfo_numbox.vue";

export default {
  data() {
    return {
      id: this.$route.params.id,
      lunbotu: [],
      goodsinfo: {},
      ballFlag: false, //控制小球显示与隐藏
      selectedCount: 1,
    };
  },
  components: {
    swiper,
    numbox
  },
  created() {
    this.getLunbotu();
    this.getGoodsInfo();
  },
  methods: {
    getLunbotu() {
      this.$http.get("api/getthumimages/" + this.id).then(result => {
        if (result.body.status === 0) {
          result.body.message.forEach(element => {
            element.img = element.src;
          });
          this.lunbotu = result.body.message;
        }
      });
    },
    getGoodsInfo() {
      this.$http.get("api/goods/getinfo/" + this.id).then(result => {
        if (result.body.status === 0) {
          this.goodsinfo = result.body.message[0];
        }
      });
    },
    goDesc(id) {
      // 点击使用编程式导航跳转到 图文介绍页面
      this.$router.push({ name: "goodsdesc", params: { id } });
    },
    goComment(id) {
      this.$router.push({ name: "goodscomment", params: { id } });
    },
    addToShopCar() {
      // 添加到购物车
      this.ballFlag = !this.ballFlag;
      // { id:商品的id, count:要购买的数量, price:商品的单价, selected:false }
      // 拼接出一个 要保存到 store 中 car 数组里的商品信息对象
      var goodsinfo = {
        id: this.id, 
        count: this.selectedCount, 
        price: this.goodsinfo.sell_price,
        selected: true 
      };
      // 调用 store 中的 mutations 来将商品加入购物车
      this.$store.commit("addToCar", goodsinfo);

    },
    beforeEnter(el){
      el.style.transform = "translate(0,0)";
    },
    enter(el,done){
      el.offsetWidth;

      // 小球动画优化思路
      // 1. 先分析导致 动画 不准确的 本质原因：我们把 小球 的最终位置写死了
      // 2. 只要分辨率与 测试的时候不一样，或者滚动条有一定的距离，就有问题
      // 3. 因此，我们经过分析，得到结论：不能把横纵坐标写死，应该动态计算这个坐标值
      // 4. 解决思路：先得到徽标 的横纵，再得到小球的横纵坐标，然后 让x，y求差，得到的结果就是
      // 5. 如何获取徽标和小球的位置？？？ domObject.getBoundingClientRect()
      // 获取小球的位置
      const ballPosition = this.$refs.ball.getBoundingClientRect();
      // 获取徽标的位置
      const badgePosition = document.getElementById("badge").getBoundingClientRect();

      const xDist = badgePosition.left - ballPosition.left;
      const yDist = badgePosition.top - ballPosition.top;

      // ES6 的模版字符串 用 ``,而不是 ""
      el.style.transform = `translate(${xDist}px,${yDist}px)`;
      el.style.transition = 'all 1s cubic-bezier(.39,.09,1,.18)'
      done()
    },
    afterEnter(el){
      this.ballFlag = !this.ballFlag;
    },
    getSelectedCount(count){
      // 子组件把 选中的 数量传递给父组件的时候，把选中的值保存到data上
      this.selectedCount = count;
      console.log("父组件拿到的数量为：" + this.selectedCount);
    },
  }
};
</script>

<style lang="scss" scoped>
.goodsinfo-container {
  background-color: #eee;
  overflow: hidden;
  .now_price {
    color: red;
    font-size: 16px;
    font-weight: bold;
  }
  .mui-card-footer {
    display: block;
    button {
      margin: 10px 0;
    }
  }
  .ball {
    width: 15px;
    height: 15px;
    border-radius: 50%;
    background-color: red;
    position: absolute;
    z-index: 99;
    top: 390px;
    left: 146px;
  }
}
</style>

