<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <home-swiper :banners="banners"/>
    <recommend-view :recommends="recommends"/>
    <feature-view></feature-view>
    <tab-control :titles="['流行','新款','精品']" class="tab-control"></tab-control>

  </div>
</template>

<script>
  import HomeSwiper from './childComps/HomeSwiper'
  import RecommendView from "./childComps/RecommendView"
  import FeatureView from "./childComps/FeatureView"

  import NavBar from 'components/common/navbar/NavBar'
  import TabControl from "components/content/tabControl/TabControl"

  import {getHomeMuLtidata,getHomeGoods} from "network/home"

  export default {
    name: "Home",
    components:{
      NavBar,
      HomeSwiper,
      RecommendView,
      FeatureView,
      TabControl
    },
    //应该是父子组件传信的知识点，在复习一下
    data(){
      return{
        banners:[],
        recommends:[],
        goods:{
          'pop':{page:0,list:[]},
          'new':{page:0,list:[]},
          'sell':{page:0,list:[]}
        }
      }
    },
    created(){
      //1.请求数据    
      this.getHomeMuLtidata()
      //2.请求商品数据
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')
    },
    methods:{
      getHomeMuLtidata(){
        getHomeMuLtidata().then(res => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list
      })
      },
        getHomeGoods(type){
         const page=this.goods[type].page+1
         getHomeGoods(type,page).then(res =>{
           /* this.goods[type].list.push(...res.data.list)
           this.goods[type].page+=1 */
           console.log(res)
      })
       } 
    }
  }
</script>

<style scoped>
  #home {
    padding-top: 44px;
  }
  .home-nav {
    background-color: var(--color-tint);
    color: white;
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 10;
  }
  .tab-control{
    position: sticky;
    top:44px;
  }
</style>
