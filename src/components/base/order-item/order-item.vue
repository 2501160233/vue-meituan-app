<!-- 订单 项目 组件 -->

<template>
  <div class="order-item">
    <div class="top">
      <img v-lazy="data.pic_url">
      <div class="name">{{data.name}}</div>
      <div class="status">已完成</div>
    </div>

    <div class="mid">

      <div class="food">
        <img style="width:30px;height:30px;border-radius:50%;" v-lazy="data.pic_url">
        <span class="food-span">香辣爆炒牛肚煲仔饭</span>
        <span style="float:right">￥12</span>
      </div>
      <div class="cost"><span>共1件</span></div>
    </div>

    <div class="bottom">
      <div class="again" @click="again">再来一单</div>
    </div>
  </div>
</template>

<script>
export default {
  components: {},
  data () {
    return {}
  },
  props: {
    data: {
      type: Object
    }
  },
  watch: {},
  methods: {
    again (){
      this.$router.push({
            path:'/restaurant',
        })
    }
  },
  filters: {},
  computed: {},
  created () {},
  mounted () {}
}
</script>

<style lang="scss" scoped>
@import '~@/assets/scss/const.scss';
@import '~@/assets/scss/mixin.scss';

.order-item {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 160px;
  padding: 10px 5px;
  color: #333;
  background-color: #fff;
  box-sizing: border-box;
  margin-bottom: 10px;
  .top {
    flex: 1;
    padding: 0 15px;
    font-size: 14px;
    height: 50px;
    line-height: 50px;
    img {
      float: left;
      width: 40px;
      height: 40px;
      margin-top: 5px;
      margin-right: 10px;
      border-radius: 50%;
    }
    .name {
      position: relative;
      float: left;
      max-width: 50%;
      color: #333;
      font-size: 16px;
      font-weight: bold;
      @include ellipsis(1);
      // &:after {
      //   position: absolute;
      //   top: 20px;
      //   right: 75px;
      //   content: '';
      //   width: 8px;
      //   height: 8px;
      //   border: 1px solid #999;
      //   border-width: 1px 1px 0 0;
      //   transform: rotate(45deg);
      // }
    }
    .status {
      float: right;
      color: #999;
    }
  }
  .mid {
    flex: 2;
    margin-left: 62px;
    margin-right: 15px;
    padding-top: 10px;
    box-sizing: border-box;
    font-size: 14px;
    color: #666;
    @include onepx('top');
    .food {
      height: 35px;
      line-height: 30px;
      font-size: 14px;
      position: relative;
      .food-span{
        display: inline-block;
        height: 30px;
        line-height: 30px;
        margin-left: 10px;
        position: absolute;
        top:0;
      }
      span {
        font-size: 14px;
        color: #151515;
        // float: right;
      }
    }
    .cost {
      float: right;
      font-size: 12px;
      line-height: 12px;
      span {
        font-size: 12px;
        color: #151515d2;
      }
    }
  }
  .bottom {
    flex: 1;
    // @include onepx('top');
    padding: 0 15px;
    font-size: 14px;
    height: 24px;
    line-height: 24px;
    .again {
      float: right;
      width: 76px;
      height: 20px;
      line-height: 20px;
      border: 1px solid #ccc;
      text-align: center;
      font-size: 14px;
      border-radius: 2px;
      // margin: 9px 0;
    }
  }
}
</style>
