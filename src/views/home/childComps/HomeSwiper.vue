<template>
  <swiper>
      <!-- 这是因为我使用了Vtur插件，然后Eslint对vue进行了检查，因为在vue2.2.0+的版本中，使用v-for时，key是必须的 -->
    <swiper-item v-for="item in banners" :key="item.id">
      <a :href="item.link">
        <img :src="item.image" @load="imageLoad">
      </a>
    </swiper-item>
  </swiper>
</template> 

<script>
import {Swiper,SwiperItem} from "components/common/swiper"

export default {
    name:'HomeSwiper',
    props:{
        banners:{
            type:Array,
            default(){
                return []
            }
        }
    },
    data(){
      return {
        isLoad:false
      }
    },
    components:{
        Swiper,
        SwiperItem
    },
    methods:{
      imageLoad(){
        if(!this.isLoad){
          this.$emit('swiperImageLoad')
          this.isLoad=true
        }
      }
    }
}
</script>

<style>

</style>