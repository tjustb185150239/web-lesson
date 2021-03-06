<template>
    <div>
        <top-bar :focusFunc="search" >
            <div slot="right" @click="goto('cart')" class="iconfont icon-gouwuchezhengpin" style="font-size: 24px"></div>
        </top-bar>
        <my-content :refreshFunc="refresh" pull>
            <a-carousel :after-change="onChange">
                <div v-for="(item,index) in homeImgs" :style="imgStyle(item)" :key="item"><h3>{{index + 1}}</h3></div>
            </a-carousel>
            <div v-for="c in categorys" :key="c.id">
                <div class="title">
                    <div class="title-left">{{c.name}}</div>    
                    <div class="title-right" @click="gotoGoodsCategory(c.id)">查看全部</div>    
                </div> 
                <h-scroll>
                    <product-card style="flex-shrink: 0;margin-right:12px;" v-for="item in c.goods" :product="item" :key="item.id" />
                </h-scroll>
            </div>
            <div class="title">
                <div class="title-left">更多内容</div>    
            </div>
            <div class="wrapper" ref="wrapper" >
                <ul class="list" ref="list">
                    <div v-for="item in moreContent" :key="item.name" class="more-content" :style="moreContentStyle(item)">
                        <div style="opacity:0.8">{{item.name}}</div>
                    </div>
                </ul>
            </div>
        </my-content>
    </div>
</template>

<script>
import TopBar from '@/components/topbar/TopBar'
import MyContent from '@/components/content/MyContent'
import ProductCard from '@/components/product/ProductCard'
import HScroll from '@/components/scroll/HScroll'
import BScroll from 'better-scroll'
import {HttpGql,ImgUrl} from '@/kits/Http'
import {getCacheVal} from '@/kits/LocalStorage'

let moreContent = [
    {
        name:"爆款",
        backgroundColor:"#F9BEAD",
        fontColor:"#D84933",
    },
    {
        name:"特价",
        backgroundColor:"#FBD96D",
        fontColor:"#B68700",
    },
    {
        name:"二手",
        backgroundColor:"#DFF8EA",
        fontColor:"#07A565",
    },
    {
        name:"拼一拼",
        backgroundColor:"#B1EAFD",
        fontColor:"#155162",
    },
]

export default {
    name:'Home',
    data(){
        return {
            categorys:[],
            homeImgs:[],
            moreContent,
        }
    },
    components:{
        TopBar,
        MyContent,
        ProductCard,
        HScroll
    },
    /*
        beforecreate  
        1.data
        2.methods
        setup //3.0语法
        created       
    */
    created(){
        this.initData() 
    },
    computed:{
        imgStyle(){
            return (url)=>{
                return url && url !== "" ? {
                    backgroundImage: `url(${url})`,
                    backgroundSize: 'cover'
                } : ""
            }
        },
        moreContentStyle(){
            return (obj)=>{
                return {
                    backgroundColor:obj.backgroundColor,
                    color:obj.fontColor,
                }
            }
        }
    },
    methods:{
        gotoGoodsCategory(categoryId){
            this.$store.commit("setGoodCategory",categoryId)
            this.$router.push({name:"goodscategory"})
        },
        refresh(){
            return this.initData()
        },
        goto(name,content){
            content ? this.$router.push({
                name,
                params:{
                    content
                }
            }) : this.$router.push({name})
        },
        search(){
            this.$router.push({path:'/search'})
        },
        onChange(a, b, c) {
            // console.log(a, b, c);
        },
        async initData(){
            let t = '["03","06"]'
            let gql = ""
            if(getCacheVal("token") && getCacheVal("token").length > 0 ){
                gql = {
                    query:`
                            {
                                homeImgs
                                categorys(type:${t}) {
                                    id
                                    dictid
                                    name
                                    goods(count:5){
                                        id
                                        name
                                        type {
                                            id
                                        }
                                        price
                                        imgpath
                                    }
                                }
                                userCart(userid:"${getCacheVal('userid')}"){
                                    id
                                    name,
                                    gooddesc,
                                    imgpath,
                                    price,
                                    countbuy,
                                    type {
                                        id
                                    }
                                }
                            }
                        `
                    }
            }else{
                gql = {
                    query:`
                            {
                                homeImgs
                                categorys(type:${t}) {
                                    id
                                    dictid
                                    name
                                    goods(count:5){
                                        id
                                        name
                                        type {
                                            id
                                        }
                                        price
                                        imgpath
                                    }
                                }
                            }
                        `
                    }
            }
            try {
                let res = await HttpGql(gql)
                for(let c of res.data.categorys){
                    c.goods = c.goods.map((item)=>{
                        item.imgpath =  ImgUrl +item.imgpath
                        return item
                    })
                }
                this.categorys = res.data.categorys
                this.homeImgs = res.data.homeImgs
                this.$store.commit("initCart",res.data.userCart ? res.data.userCart.map((item)=>{
                    item.imgpath = ImgUrl + item.imgpath
                    return item
                }) : [])
                return true
            } catch (error) {
                let goods = []
                for(let item of [1,2,3,4,5]){
                    goods.push({
                        id:item,
                        name:"产品名称",
                        price:0
                    })
                    this.homeImgs.push("")
                }
                this.categorys.push({
                    name:"商品类别",
                    goods
                })
                return false
            } 
        }
    },
    mounted() {
        this.$nextTick(() => { // 使用 this.$nextTick 为了确保组件已经渲染完毕
            let itemWidth = 138 // 这里是设置列表每一项的宽度
            let margin = 0
            // width是整个列表的宽度
            let width = (itemWidth + margin) * 5 - margin
            // console.log(width)
            this.$refs.list.style.width = width + 'px' // 设置.list的宽度的宽度
            this.$nextTick(() => {
                if (!this.picScroll) {
                    this.picScroll = new BScroll(this.$refs.wrapper, {
                        scrollX: true,
                        eventPassthrough: 'vertical' // 忽略竖直方向的滚动
                    })
                } else {
                    this.picScroll.refresh()
                }
            })
        })
    }
}
</script>

<style scoped>
.ant-carousel >>> .slick-slide {
  text-align: center;
  height: 160px;
  line-height: 160px;
  background: #e5e5e5;
  overflow: hidden;
  border-radius: 15px;
}

.ant-carousel >>> .slick-slide h3 {
  color: #fff;
}

.wrapper {
    overflow: hidden;
    white-space: nowrap; /*当子元素超过父元素宽度的时候，不会折行*/
    margin-top:12px;
    touch-action: none;
}

.wrapper .list {
    display: flex;
    padding:0px;
}

.title {
    display: flex;
    justify-content: space-between;
    margin-top:24px;
}

.title-left {
    font-size: 14px;
    font-weight: bold;
    color: rgb(0 0 0 / 0.5)
}

.title-right {
    font-size: 12px;
    color:#B620E0;
    align-self: flex-end;
}

.more-content {
    flex-shrink: 0;
    margin-right: 12px;
    width:119px;
    height:90px;
    border-radius:15px;
    background-color:#E5E5E5;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 19px;
    font-weight: bold;
}


</style>