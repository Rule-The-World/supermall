<template>
    <div class="tab-control">
        <div v-for="(item,index) in titles" 
        :key="item.id" 
        class="tab-control-item"
        :class="{active: index==currentIndex}"
        @click="itemClick(index)">
            <span>{{item}}</span>
        </div>
    </div>
</template>

<script>
export default {
    name:"TabControls",
    props:{
        titles:{
            type:Array,
            default(){
                return []
            }
        }
    },
    data() {
        return{
            currentIndex: 0
        }
    },
    methods:{
        itemClick(index){
            this.currentIndex=index;
            //子组件向父组件传值
            this.$emit('tabClick',index)
        }
    }
}
</script>

<style>
    .tab-control {
        display: flex;
        text-align: center;
        font-size: 15px;
        line-height: 40px;
        background-color: #fff;
    }
    .tab-control-item {
        flex: 1;     
    }
    .active span{
        color: var(--color-height-text);
        border-bottom: 3px solid var(--color-tint);
        padding: 5px;
    }
</style>