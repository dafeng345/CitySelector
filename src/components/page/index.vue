<template>
    <div class="index">
        <div class="intro">
            选择城市前，会自动定位当前城市<br/>
            选择城市后，会取消对当前城市的自动定位
        </div>
        <div class="address-bar">
            <div class="address-choose">
                <span class="address-choose-title">选择地址</span>
                <router-link tag="span" :to="{name: 'cityList'}" class="address-choose-content">{{cityName}}<span
                    class="address-choose-icon">丨</span></router-link>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        components: {},
        props: ['cityName', 'shouldChangeCity'],
        data() {
            return {
            };
        },
        created() {
        },
        mounted() {
            this.$nextTick(() => {
                this.baiduLocation();
            });
        },
        methods: {
            /**
             * [百度地图定位]
             * @param  {[type]}       [无入参]
             * @return {[type]}       [无返回值]
             */
            baiduLocation() {
                this.$parent.shouldChangeCity = true;
                let _that = this;
                /*eslint-disable*/
                // 百度地图定位
                var map = new BMap.Map("allmap");
                var point = new BMap.Point(116.331398, 39.897445);
                map.centerAndZoom(point, 12);
                var geoc = new BMap.Geocoder();
                var geolocation = new BMap.Geolocation();
                geolocation.getCurrentPosition(function (r) {
                    if (this.getStatus() === BMAP_STATUS_SUCCESS) {
                        console.log('定位成功');
                        console.log(r.point);
                        geoc.getLocation(r.point, function (rs) {
                            var addComp = rs.addressComponents;
                            console.log(addComp.province + ", " + addComp.city + ", " + addComp.district + ", " + addComp.street + ", " + addComp.streetNumber);
                            if (_that.shouldChangeCity) {
                                _that.$parent.cityName = addComp.city.replace('市', '');
                            }
                        });
                    }
                    else {
                        console.log('failed' + this.getStatus());
                    }
                }, {enableHighAccuracy: true});
                /*eslint-disable*/
            }
        },
        beforeRouteEnter(to,from,next){
            // debugger;
            next(vm=>{
                console.log(vm,vm.shouldChangeCity,'index-route');
            })
            
        }
    };
</script>

<style lang="scss" scoped>
     @function rem($px) {
        @return $px * (1/100) * 1rem;
    }
    .index {
        width: 100%;
        overflow-x: hidden;
        .intro {
            background-color: white;
            padding: rem(20);
            font-size: rem(18);
        }
        .address-bar {
            padding: 0 rem(20);
            background-color: #FFFFFF;
            border-top: #eeeeee solid rem(2);
            .address-choose {
                height: rem(68);
                line-height: rem(68);
                display: flex;
                font-size: rem(24);
                .address-choose-title {
                    flex: 1;
                    font-size: rem(24);
                    color: #111111;
                    font-weight: 500;
                    text-align: left;
                }
                .address-choose-content {
                    flex: 1;
                    color: #999999;
                    text-align: right;
                    .address-choose-icon {
                        font-family: 'iconfont';
                        padding-left: rem(20);
                    }
                }
            }
        }
    }
</style>
