<template>
  <div id="home" class="wrapper">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control :titles="['流行','新款','精品']" 
       @tabClick="tabClick" ref="tabControl1" 
       class="tab-control" v-show="isTabFixed"/>
    <scroll class="content" 
    ref="scroll" :probe-type="3" 
    @scroll="contentScroll"
    @pullingUp="loadMore"
    :pull-up-load="true">
      <home-swiper :banners="banners" 
      @swiperImageLoad="swiperImageLoad"/>
      <recommend-view :recommends="recommends"/>
      <feature-view/>
      <tab-control :titles="['流行','新款','精品']" 
       @tabClick="tabClick" ref="tabControl2"/>
      <goods-list :goods="showGooods"/>
    </scroll>


    <!-- 组件监听事件必须加'.native' -->
    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
    
  </div>
</template>

<script>
  import HomeSwiper from './childComps/HomeSwiper'
  import RecommendView from "./childComps/RecommendView"
  import FeatureView from "./childComps/FeatureView"

  import NavBar from 'components/common/navbar/NavBar'
  import TabControl from "components/content/tabControl/TabControl"
  import GoodsList from "components/content/goods/GoodsList"
  import Scroll from 'components/common/scroll/Scroll'
  import BackTop from 'components/content/backTop/BackTop'

  import {getHomeMuLtidata,getHomeGoods} from "network/home"
  import {debounce} from "common/utils"

  export default {
    name: "Home",
    components:{
      NavBar,
      HomeSwiper,
      RecommendView,
      FeatureView,
      TabControl,
      GoodsList,
      Scroll,
      BackTop
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
        },
        currentType: 'pop',
        isShowBackTop: false,
        tabOffsetTop: 0,
        isTabFixed:false,
        saveY: 0,

      }
    },
    computed:{
      showGooods(){
        return this.goods[this.currentType].list
      }
    },
    activated(){
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
      this.$refs.scroll.refresh()
      console.log(this.saveY)
      
    },
    deactivated(){
      //1.保存y值
      this.saveY= this.$refs.scroll.getScrollY()
      //console.log(this.saveY)
      //2.取消全局事件的监听
      this.$bus.$off('itemImageLoad',)
    },
    created(){
      //1.请求数据    
      this.getHomeMuLtidata()
      //2.请求商品数据
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')
  
    },
    mounted(){
      //1.监听item中图片加载完成
      let refresh =debounce(this.$refs.scroll.refresh)

      this.$bus.$on('itemImageLoad' , ()=> {
         refresh()
      })

      },
    methods:{
      /* 事件监听相关方法 */
      tabClick(index){
        switch (index) {  
          case 0:
            this.currentType="pop"
            break;
          case 1:
            this.currentType="new"
            break;
          case 2:
            this.currentType="sell"
            break;
        }
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl2.currentIndex = index;
      },
      //返回顶部
      backClick(){
        this.$refs.scroll.scrollTo(0,0)
      }, 
      //滚动位置监听
      contentScroll(position){
        //1.判断BackTo是否显示
        this.isShowBackTop=(-position.y)>1000
        //2.决定tabControl是否吸顶(position:fixed)
        this.isTabFixed = (-position.y)>this.tabOffsetTop
      },
      //上拉加载更多
      loadMore(){
        this.getHomeGoods(this.currentType)
      }, 
      swiperImageLoad(){
        this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
      },
      /* 网络请求相关方法 */
      getHomeMuLtidata(){
        getHomeMuLtidata().then(res => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list
      })
      },
      getHomeGoods(type){
         const page=this.goods[type].page+1
         getHomeGoods(type,page).then(res =>{
           //对list数组进行解析并把内容push到goods中的list数组中
           this.goods[type].list.push(...res.data.list)
           this.goods[type].page+=1 
           //console.log(res)
           //完成下拉加载
          this.$refs.scroll.finishPullUp()
      }) 
      },
     

    }
  }
</script>
<!-- scoped可理解为作用域只在当前文件有效 -->
<style scoped>
  #home {
    height: 100vh;
    position: relative;
  }
  .home-nav {
    background-color: var(--color-tint);
    color: white;
  }
  .tab-control{
    position: relative;
    z-index: 9;
  }
  .content{
    overflow: hidden; 
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
</style>
