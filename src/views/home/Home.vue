<!--  -->
<template>
    <div id="home">
        <nav-bar class="home-nav">
            <div slot="center">购物街</div>
        </nav-bar>

        <scroll class="content" 
            ref="scroll" 
            :probe-type="3" 
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
            <home-swiper :banners="banners"/>
            <recommend-view :recommends="recommends"/>
            <feature-view/>
            <tab-control class="tab-control" :titles="['流行','新款','精选']"
                @tabClick="tabItemClick"/>
            <goods-list :goods="showGoods"/>        
        </scroll>
        
        <back-top @click.native="backClick" v-show="isShowBackTop"/>
        </div>
</template>

<script>
    //这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
    //例如：import 《组件名称》 from '《组件路径》';
    import HomeSwiper from "./childComps/HomeSwiper.vue"
    import RecommendView from "./childComps/RecommendView.vue"
    import FeatureView from "./childComps/FeatureView.vue"
    
    import NavBar from 'components/common/navbar/NavBar'
    import TabControl from "components/content/tabControl/TabControl" 
    import GoodsList from "components/content/goods/GoodsList"
    import Scroll from 'components/common/scroll/Scroll'
    import BackTop from 'components/content/backtop/BackTop'

    import { getHomeMultidata, getHomeGoods } from "network/home"


    export default {
    //import引入的组件需要注入到对象中才能使用
        name: 'Home',
        components: {
            HomeSwiper,
            RecommendView,
            FeatureView,
            NavBar,
            TabControl,
            GoodsList,
            Scroll,
            BackTop
        },
        data() {
            //这里存放数据
            return {
                banners: [],
                recommends: [],
                goods: {
                    'pop': {page: 1, list: []},
                    'new': {page: 0, list: []},
                    'sell': {page: 0, list: []}
                },
                currentType: 'pop',
                isShowBackTop: false
            };
        },
        //监听属性 类似于data概念
        computed: {
            showGoods() {
                return this.goods[this.currentType].list;
            }
        },
        //监控data中的数据变化
        watch: {},
        //方法集合
        methods: {
            // 事件监听相关的方法
            tabItemClick(index) {
                switch (index) {
                    case 0:
                        this.currentType = 'pop'
                        break;
                    case 1: 
                        this.currentType = 'new'
                        break;
                    case 2:
                        this.currentType = 'sell'
                        break;
                    // default:
                    //     break;
                }
            },
            backClick() {
                this.$refs.scroll.scroll.scrollTo(0, 0, 500) //x坐标, y坐标, t时间毫秒            },
            },
            contentScroll(position) {
                this.isShowBackTop = -position.y > 1000 ? true : false 
            },
            loadMore() {
                this.getHomeGoods(this.currentType)
            
                this.$refs.scroll.scroll.refresh()
            },

            // 网络请求相关的方法
            getHomeMultidata() {
                getHomeMultidata().then(res => {
                    this.banners = res.data.banner.list;
                    this.recommends = res.data.recommend.list;
                })    
            },
            getHomeGoods(type) {
                const page = this.goods[type].page + 1;
                getHomeGoods(type, page).then(res => {
                    this.goods[type].list.push(...res.data.list)
                    this.goods[type].page += 1;

                    this.$refs.scroll.scroll.finishPullUp()
                })    
            }
            
        },
        //生命周期 - 创建完成（可以访问当前this实例）
        created() {
            // 1.请求多个数据
            this.getHomeMultidata()

            // 2.请求商品数据
            this.getHomeGoods('pop')
            this.getHomeGoods('new')
            this.getHomeGoods('sell')
        },
        //生命周期 - 挂载完成（可以访问DOM元素）
        mounted() {
        
        },
        beforeCreate() {}, //生命周期 - 创建之前
        beforeMount() {}, //生命周期 - 挂载之前
        beforeUpdate() {}, //生命周期 - 更新之前
        updated() {}, //生命周期 - 更新之后
        beforeDestroy() {}, //生命周期 - 销毁之前
        destroyed() {}, //生命周期 - 销毁完成
        activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
    }
</script>
<style scoped>
    #home {
        height: 100vh;
        position: relative;
    }

    .home-nav {
        background-color: var(--color-tint);
        color: white;
        text-align: center;
        position: fixed;
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        z-index: 9;

    }

    .tab-control {
        position: sticky;
        top: 44px;
        z-index: 9;
    }

    .content {
        /* height: calc(100% - 93px);
        margin-top: 44px; */

        position: absolute;
        top: 44px;
        bottom: 49px;
        left: 0;
        right: 0;

        overflow: hidden;
    }

</style>