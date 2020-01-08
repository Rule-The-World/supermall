<template>
  <div id="detail">
      <detail-nav-bar class="detail-nav"/>
      <scroll class="content" ref="scroll">
        <detail-swiper :top-images="topImages"/>
        <detail-base-info :goods="goods"/>
        <detail-shop-info :shop="shop"/>
        <detail-goods-info :detail-info="detailInfo" 
        @imageLoad="imageLoad"/>
        <detail-param-info :param-info="paramInfo"/>
        <detail-comment-info :comment-info="commentInfo"/>
        <goods-list :goods="recommends"/>

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
    },
    methods:{
        imageLoad(){
            this.$refs.scroll.refresh()
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