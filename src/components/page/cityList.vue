<template>
  <div class="cityList" ref="citylistpage">
    <!-- better-scroll 只处理容器（wrapper）的第一个子元素（content）的滚动，其它的元素都会被忽略。 -->
    <div>
        <!-- 搜索栏开始 -->
      <div class="search-bar">
        <div class="search-input-bar">
          <!-- 搜索提示框中 搜索图标 -->
          <div v-show="!showCancelIcon" class="search-icon">匨</div>
          <!-- 搜索输入框 -->
          <form action="javascript: return false;">
            <input
              type="search"
              autocomplete="off"
              @keyup.enter="keyupSearch(searchMsg)"
              placeholder="请输入城市名"
              v-model="searchMsg"
            />
          </form>
          <!-- 搜索提示框中 关闭图标 -->
          <div
            class="close-icon"
            @click="cancelSearch()"
            v-show="showCancelIcon"
          >
            慶
          </div>
        </div>
      </div>
      <!-- 搜索栏结束 -->
      <!-- 当搜索框没有输入内容时，展示城市列表 -->
      <div class="allcity-bar" v-if="!showSearchInfo" ref="allcitybar">
        <div
          class="city-item-list"
          v-for="(list, word, i) in cityList"
          :key="i"
        >
          <div class="city-title">{{ word }}</div>
          <ul>
            <li
              @click="selectCity(city.name)"
              class="city-item"
              v-for="(city, index) in list"
              :key="index"
            >
              {{ city.name }}
            </li>
          </ul>
        </div>
      </div>
      <!-- 当在搜索框中输入内容时，展示出的搜索结果。 -->
      <div class="city-search-bar" v-if="showSearchInfo">
        <ul>
          <li
            @click="selectCity(city.name)"
            class="city-item"
            v-for="(city, index) in searchRes"
            :key="index"
          >
            {{ city.name }}
          </li>
        </ul>
      </div>
    </div>
    <div class="fast-sel-bar" v-show="!showSearchInfo">
      <p
        class="fast-sel-item"
        :class="{ fastSelActive: index === curCityListIndex }"
        v-for="(list, word, index) in cityList"
        :data-index="index"
        @touchstart="onSelectFastTouchStart"
        @touchmove.stop.prevent="onSelectFastTouchMove"
        :key="index"
      >
        {{ word }}
      </p>
    </div>
  </div>
</template>

<script>
import { cityList } from "@/common/city.js";
import BScroll from "better-scroll";
export default {
  props: ["shouldChangeCity"],
  data() {
    return {
      searchMsg: "", // 搜索的信息
      cityList: {}, // 所有城市
      showSearchInfo: false, // 是否显示搜索提示框（默认不显示搜索出来的内容），
      searchRes: [], // 搜索结果
      listHeight: [], // 每组城市的高度
      scrollY: 0, // Y方向距离
      touch: {}, //
      cityDomList: null, // 城市dom
      htmlFontSize: 0, // 用于计算高度
      curCityListIndex: 0, // 当前快速选择index
    };
  },
  created() {
    // 获得axios请求的数据
    this.cityList = cityList;
    let that = this;
    // setTimeout(() => {
    this.$nextTick(() => {
      that._initScroll();
      that.calculateHeight();
    });
    // }, 20);
  },
  mounted() {
    let that = this;
    this.cityDomList = this.$refs.allcitybar.getElementsByClassName(
      "city-item-list"
    );
    setTimeout(() => {
      that.htmlFontSize = window.document.getElementsByTagName(
        "html"
      )[0].style.fontSize;
    }, 20);
  },
  computed: {
    // 计算是否显示搜索提示框
    showCancelIcon() {
      return this.searchMsg !== "";
    },
  },
  watch: {
    scrollY(newY, oldY) {
      const listHeight = this.listHeight;
      console.log(newY, oldY);
      // 当滚动到顶部，newY>0
      if (newY > 0) {
        this.curCityListIndex = 0;
        return;
      }
      // 在中间部分滚动
      for (let i = 0; i < listHeight.length - 1; i++) {
        let height1 = listHeight[i];
        let height2 = listHeight[i + 1];
        if (-newY >= height1 && -newY < height2) {
          this.curCityListIndex = i;
          this.diff = height2 + newY;
          return;
        }
      }
      // 当滚动到底部，且-newY大于最后一个元素的上限;
      this.currentIndex = listHeight.length - 2;
    },
    // 观测搜索框内输入的文字，监听搜索框内容改变时触发
    searchMsg() {
      if (this.searchMsg) {
        // 重新计算 better-scroll
        this.cityListScroll.refresh();
      }
      this.searchRes = [];
      // 事件截留
      if (this.searchMsgTimeoutFlag) {
        window.clearTimeout(this.searchMsgTimeoutFlag);
      }
      this.searchMsgTimeoutFlag = window.setTimeout(() => {
        if (this.searchMsg !== "") {
          this.showSearchInfo = true;
          this.searchCityByName(this.searchMsg);
          this.cityListScroll.refresh();
        } else {
          this.showSearchInfo = false;
        }
      }, 200);
    },
    showSearchInfo() {
      this.$nextTick(() => {
        this.cityListScroll.refresh();
      });
    },
  },
  methods: {
    /**
     * [选择城市]
     * @param  {[type]}         [无入参]
     * @return {[type]}         [无返回值]
     */
    selectCity(cityName) {
      this.$parent.cityName = cityName;
      this.$parent.shouldChangeCity = false;
      this.$router.go(-1);
    },
    /**
     * [退出文字搜索(输入框X号功能)]
     * @param  {[type]}         [无入参]
     * @return {[type]}         [无返回值]
     */
    cancelSearch() {
      this.searchMsg = "";
    },
    /**
     * [keyup事件的触发，当输入框的内容包含于城市列表时触发城市选择]
     * @param  {[string]} searchMsg  [搜索框内的文字]
     * @return {[type]}              [无返回值]
     */
    keyupSearch(searchMsg) {
      if (this.searchRes.length > 0 && searchMsg === this.searchRes[0].name) {
        this.selectCity(searchMsg);
      }
      return false;
    },
    /**
     * [根据输入的文字在所有城市中检索]
     * @param  {[string]} cityName  [搜索框内的文字]
     * @return {[type]}             [无返回值]
     */
    searchCityByName(cityName) {
      let hasChinese = /.*[\u4e00-\u9fa5]+.*$/.test(cityName); // 是否有中文
      let hasWord = /^[a-zA-Z]/.test(cityName); // 是否有英文
      if (hasChinese && hasWord) {
        return;
      } else if (hasChinese) {
        for (let i in cityList) {
          for (let m of cityList[i]) {
            if (m.name.indexOf(cityName) > -1) {
              this.searchRes.push(m);
            }
          }
        }
      } else if (hasWord) {
        for (let i in cityList) {
          for (let m of cityList[i]) {
            if (m.pinyin.indexOf(cityName) > -1) {
              this.searchRes.push(m);
            }
          }
        }
      }
    },
    /**
     * [初始化scroll]
     * @param  {[type]}         [无入参]
     * @return {[type]}         [无返回值]
     */
    _initScroll() {
      this.cityListScroll = new BScroll(this.$refs.citylistpage, {
        click: true,
        probeType: 3,
      });
      this.cityListScroll.on("scroll", (pos) => {
        this.scrollY = pos.y;
      });
    },
    /**
     * [计算每个城市区块的高度]
     * @param  {[type]}         [无入参]
     * @return {[type]}         [无返回值]
     */
    calculateHeight() {
      this.$nextTick(() => {
        let cityList = this.$refs.allcitybar.getElementsByClassName(
          "city-item-list"
        );
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < cityList.length; i++) {
          let item = cityList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      });
    },
    /**
     * [跳转到快速检索对应首字母的位置]
     */
    onSelectFastTouchStart(e) {
      let anchorIndex = e.target.getAttribute("data-index");
      let firstTouch = e.touches[0];
      this.touch.y1 = firstTouch.pageY;
      this.touch.anchorIndex = anchorIndex;
      let el = this.cityDomList[anchorIndex];

      this.scrollY = -this.listHeight[anchorIndex];
      console.log(this.scrollY);
      this.cityListScroll.scrollToElement(el);
    },
    /**
     * [滑动到快速检索对应首字母的位置]
     */
    onSelectFastTouchMove(e) {
      let firstTouch = e.touches[0]; //
      this.touch.y2 = firstTouch.pageY;
      let itemHeight = 0.2 * parseInt(this.htmlFontSize); // 高度为0.2rem，换算为实际高度
      // let itemHeight = 20; // 高度为0.2rem，换算为一个fast-sel-item的实际高度
      // 计算快速滑动了几项
      let delta = ((this.touch.y2 - this.touch.y1) / itemHeight) | 0;
      // console.log(delta,this.touch.y2,this.touch.y1,itemHeight);
      let anchorIndex = parseInt(this.touch.anchorIndex) + delta;
      let el = this.cityDomList[anchorIndex];
      // better-scroll向上滑动时scrollY取负值
      this.scrollY = -this.listHeight[anchorIndex];
      this.cityListScroll.scrollToElement(el);
    },
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      console.log(vm, vm.shouldChangeCity, "cityList-route");
    });
  },
};
</script>

<style lang="scss">
@function rem($px) {
  @return $px * (1/100) * 1rem;
}
.cityList {
  width: 100%;
  position: absolute;
  overflow: hidden;
  .search-bar {
    /*position: fixed;*/
    z-index: 10;
    background-color: #ffffff;
    width: 100%;
    height: rem(66);
    border-bottom: solid rem(1) #f0f0f0;
    .search-input-bar {
      position: relative;
      top: rem(12);
      display: flex;
      height: rem(42);
      line-height: rem(42);
      margin: 0 rem(20);
      padding: 0 rem(27);
      background-color: #eeeeee;
      font-size: rem(20);
      border-radius: rem(27);
      .search-icon,
      .close-icon {
        flex: 0 0 auto;
        font-family: iconfont;
        font-size: rem(25);
        color: #999999;
      }
      form {
        flex: 1 0 auto;
        width: 100%;
        input {
          position: relative;
          top: rem(-3);
          outline: none;
          appearance: textfield;
          color: #999999;
          background-color: #eeeeee;
          text-indent: rem(5);
          width: 100%;
        }
        input[type="search"]::-webkit-search-cancel-button {
          display: none;
        }
      }
    }
  }
  .allcity-bar {
    width: 100%;
    background-color: #ffffff;
    .city-item-list {
      .city-title {
        padding-left: rem(10);
        height: rem(30);
        line-height: rem(30);
        font-size: rem(20);
        color: #666666;
        background-color: #eeeeee;
      }
    }
  }
  .fast-sel-bar {
    position: fixed;
    top: rem(80);
    right: 0;
    width: rem(30);
    text-align: center;
    .fast-sel-item {
      height: rem(20);
      line-height: rem(20);
      color: #999999;
      font-size: rem(14);
    }
    .fastSelActive {
      color: #f28300;
    }
  }
  ul {
    padding: 0 rem(10);
    font-size: rem(20);
    background-color: #ffffff;
    .city-item {
      padding-left: rem(11);
      height: rem(40);
      line-height: rem(40);
      font-size: rem(16);
      border-bottom: rem(1) solid #eeeeee;
    }
  }
}
</style>
