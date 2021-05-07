<template>
    <div id="app">
        <!--allmap->地图容器，无内容-->
        <div id="allmap"></div>
        <v-header></v-header>
        <keep-alive>
            <!-- /路径下存放index组件，/cityList路径下存放cityList组件 -->
            <router-view :shouldChangeCity="shouldChangeCity" :cityName="cityName" :curCompany="curCompany" class="app-router"></router-view>
        </keep-alive>
    </div>
</template>

<script>
    import header from '@/components/header/header.vue';
    export default {
        name: 'app',
        data() {
            return {
                cityName: '正在定位',
                shouldChangeCity: true,
                headerTitle: '水费缴费',
                curCompany: {},
                backType: '',
                showSubTitle: false
            };
        },
        created() {
//            this.$util.forbidScal();
        },
        // beforeMount() {

        //      var html= document.documentElement;
        //      var widths= html.clientWidth;
        //      var num= 10;
        //      html.style.fontSize= widths/num+'px';
        // },
        components: {
            'v-header': header
        }
        

    };
</script>

<style lang="scss">
    @charset "utf-8";
    @import "../static/css/base.css";
    
    @function rem($px) {
        @return $px*(1/100)*1rem;
    }
    #app {
        .app-router {
            margin-top: rem(68);
            // margin-top:68/$rem+'rem';
            position: absolute;
            bottom: 0;
            top: 0;
            background-color: #F5F4F9;
        }
    }
</style>
