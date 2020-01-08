<template>
  <div id="detail">
      <detail-nav-bar class="detail-nav" 
      @titleClick="titleClick" ref="nav"/>
      <scroll class="content" ref="scroll" 
      :probe-type="3" @scroll="contentScroll">
        <detail-swiper :top-images="topImages"/>
        <detail-base-info :goods="goods"/>
        <detail-shop-info :shop="shop"/>
        <detail-goods-info :detail-info="detailInfo" 
        @imageLoad="imageLoad"/>
        <detail-param-info ref="params" :param-info="paramInfo"/>
        <detail-comment-info ref="comment" :comment-info="commentInfo"/>
        <goods-list ref="recommend" :goods="recommends"/>

      </scroll>
  </div>
</template>

<script>
import DetailNavBar from "./childComps/DetailNavBar"
import DetailSwiper from "./childComps/DetailSwiper"
import DetailBaseInfo from "./childComps/DetailBaseInfo"
import DetailShopInfo from "./childComps/DetailShopInfo"
import DetailGoodsInfo from "./childComps/DetailGoodsInfo"
import DetailParamInfo from "./childComps/DetailParamInfo"
import DetailCommentInfo from "./childComps/DetailCommentInfo"
import GoodsList from "components/content/goods/GoodsList"

import Scroll from "components/common/scroll/Scroll"

import {getDetail,getRecommend,Goods,Shop,GoodsParam} from "network/detail"
import {debounce} from "common/utils"

export default {
    name:'Detail',
    components:{
        DetailNavBar,
        DetailSwiper,
        DetailBaseInfo,
        DetailShopInfo,
        Scroll,
        DetailGoodsInfo,
        DetailParamInfo,
        DetailCommentInfo,
        GoodsList
    },
    data(){
        return {
            iid:null,
            topImages:[],
            goods:{},
            shop:{},
            detailInfo:{},
            paramInfo:{},
            commentInfo:{},
            recommends:[],
            themeTopYs:[],
            getThemeTopY: null,
            currentIndex: 0,


        }
    },
    created(){
        //1.保存传入的iid
        this.iid = this.$route.params.iid
        //2.根据iid请求数据详情
        getDetail(this.iid).then(res=>{
            //console.log(res)
            const data = res.result;
            //获取顶部轮播图片
            this.topImages = data.itemInfo.topImages
            //获取商品信息
            this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services)
            //获取店铺信息
            this.shop = new Shop(data.shopInfo)
            //获取商品的详情数据
            this.detailInfo = data.detailInfo
            //获取参数信息
            this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule)
            //获取评论信息
            if(data.rate.cRate != 0){
                this.commentInfo = data.rate.list[0]
            }

        })
        //3.请求推荐数据
        getRecommend().then(res=>{
            this.recommends=res.data.list
        })
        //4.给getThemeTopY复制(对给this.themeTopYs赋值的操作进行防抖)
        this.getThemeTopY = debounce(()=>{
            this.themeTopYs=[]
            this.themeTopYs.push(0)
            this.themeTopYs.push(this.$refs.params.$el.offsetTop)
            this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
            this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
            //console.log(this.themeTopYs)
        },100)
    },
    methods:{
        imageLoad(){
            this.$refs.scroll.refresh()
            this.getThemeTopY()
            

        },
        titleClick(index){
            //console.log(index)
            this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200)
            
        },
        contentScroll(position){
            //console.log(position)
            const positionY = -position.y
            let length = this.themeTopYs.length
            for (let i = 0;i < length;i++) {
                if(this.currentIndex !== i && ((i < length - 1 && 
                positionY >= this.themeTopYs[i]
                && positionY < this.themeTopYs[i+1]) || 
                (i === length - 1 && positionY >= this.themeTopYs[i]))){
                    this.currentIndex = i;
                    console.log(this.currentIndex)
                    this.$refs.nav.currentIndex = this.currentIndex
                }
            }
        }
    }
}
</script>

<style scoped>
    #detail {
        position: relative;
        z-index: 9;
        background-color: #fff;
        height: 100vh;
    }
    .content {
        height: calc(100% - 44px)
               
    }
    .detail-nav {
        position: relative;
        z-index: 9;
        background-color: #fff;
    }
</style>