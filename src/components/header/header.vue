<template>
  <div class="v-header header">
    <div class="header-content">
      <div class="back" @click="goBack()" v-if="$route.meta.hasBack">
        <span class="back-icon">亃</span>返回
      </div>
      <div class="title">城市选择示例</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      lastUrl: ""
    };
  },
  methods: {
    goBack() {
      this.$router.go(-1);
    }
  }
};
</script>

<style lang="scss">
@function rem($px) {
  @return $px * (1/100) * 1rem;
}
.v-header {
  position: fixed;
  top: 0;
  z-index: 20;
  width: 100%;
  border-bottom: solid rem(1) #eeeeee;
  background-color: #f28300;
  height: rem(68);
  font-size: rem(22);
  font-family: "Micorsoft Yahei";
  color: #ffffff;
  .header-content {
    position: absolute;
    width: 100%;
    bottom: 0;
    height: rem(68);
    line-height: rem(68);
    .back {
      position: absolute;
      left: rem(20);
      font-size: rem(18);
      width: rem(110);
      .back-icon {
        font-family: iconfont;
      }
    }
    .sub-title {
      position: absolute;
      right: rem(20);
      bottom: 0;
      font-size: rem(30);
    }
    .title {
      text-align: center;
    }
  }
}
</style>
