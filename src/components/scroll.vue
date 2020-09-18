<style scoped lang="less">
    .tlstScroll{
        position: relative;
        height: 100%;
        overflow: scroll;
    }
    .tlstScroll-con{    
        position: relative;
        list-style: none;
        width: 100%;
        top: 0px;
    }
    .tlstScroll-text{ position: relative; top: -80px;}

    .tlstScroll-refresh-svg-icon{ width: 40px; height: 40px;
       background: #fff; border-radius: 50%;
       position: fixed;
       left: 50%;
       transform: translateX(-50%);
       top: 0px;
       z-index: 999;
       box-shadow: 0 3px 1px -2px rgba(0,0,0,.2), 0 2px 2px 0 rgba(0,0,0,.14), 0 1px 5px 0 rgba(0,0,0,.12);
       .box;
   }
    .tlstScroll-refresh-svg-icon svg{
        position: absolute;
        left: 50%; top: 50%;
        transform: translate(-50%,-50%) rotate(0deg);
        width:28px;height:28px;fill:currentColor;-webkit-user-select:none;-moz-user-select:none;-ms-user-select:none;user-select:none;
        
    }


.circle{animation:right-spin 1333ms cubic-bezier(.4,0,.2,1) infinite both;

    width: 24px;
    height: 24px;
    border-style: solid;
    border-color: inherit;
    border-bottom-color: transparent!important;
    border-radius: 50%;
    border-width: 2px;
    
}
@keyframes right-spin{
    0%{-webkit-transform:rotate(-130deg);transform:rotate(-130deg)}
    50%{-webkit-transform:rotate(5deg);transform:rotate(5deg)}
    to{-webkit-transform:rotate(-130deg);transform:rotate(-130deg)}
}

.refreshEnd{animation:scale0 0.5s;}
@keyframes scale0{
    0%{transform: translateX(-50%) scale(1);}
    100%{transform: translateX(-50%) scale(0);}
}

    


</style>
<template>
<div ref="tlstScroll" class="tlstScroll">
    <div ref="svgIcon" class="tlstScroll-refresh-svg-icon"
        :style="{opacity:opacity,top:top+'px'}" :class="{refreshEnd:refreshEnd}"
    >
        <template v-if="!refreshEnd">
            <svg v-if="!refreshing" viewBox="0 0 24 24" :style="{transform:transformStr}">
                <path d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z" />
            </svg>
            <svg  v-else xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="30px" height="30px" viewBox="0 0 50 50" style="enable-background:new 0 0 50 50" xml:space="preserve">
                <path fill="#FF6700" d="M43.935,25.145c0-10.318-8.364-18.683-18.683-18.683c-10.318,0-18.683,8.365-18.683,18.683h4.068c0-8.071,6.543-14.615,14.615-14.615c8.072,0,14.615,6.543,14.615,14.615H43.935z" transform="rotate(275.098 25 25)">
                    <animateTransform attributeType="xml" attributeName="transform" type="rotate" from="0 25 25" to="360 25 25" dur="0.6s" repeatCount="indefinite"></animateTransform>
                </path>
            </svg>
        </template>
    </div>
    <slot></slot>
    <infinite-loading v-if="infiniteLoadingShow && isMore" @infinite="infiniteHandler">
        <div class="infinite-no-more" slot="no-more"></div>
        <div class="infinite-no-more" slot="no-results"></div>
    </infinite-loading>
</div>
</template>

<script>
import InfiniteLoading from 'vue-infinite-loading';
export default {
   props:{
       isScrollTop:{//是否需要实时滚动高度
           type:Boolean,
           default:false,
       },
       isMore:{//是否需要上拉加载更多
           type:Boolean,
           default:false, 
       }
   },
   data(){
       return{
           num:0,
           isLoad:false,
           start:0,
           refresh:false,
           refreshing:false,
           refreshEnd:false,
           isLoad:false,
           text:"刷新",
           length:100,
           initY:-68,
           transformStr:'translate(-50%,-50%) rotate(0deg)',
           opacity:0,
           top:0,
           rotate:0,
           ismove:false,
           initTimeFu:"",
           istouchend:false,
           istouchcancel:false,
           infiniteLoadingShow:false,
           scrollTop:0,
       }
   },
   components:{
       InfiniteLoading
   },
   created(){
   },
   mounted(){
       if(this.isScrollTop){
           this.$refs.tlstScroll.addEventListener('scroll',this.scrollFu,false)
       }
       this.$refs.tlstScroll.addEventListener('touchstart',this.touchstartFu,false)
       this.$refs.tlstScroll.addEventListener('touchmove',this.touchmoveFu,false)
       this.$refs.tlstScroll.addEventListener('touchend',this.touchendFu,false)
       this.$refs.tlstScroll.addEventListener('touchcancel',this.touchcancelFu,false)
       this.infiniteLoadingShow = true;
   },
   methods:{
       scrollFu(){
           this.scrollTop = this.$refs.tlstScroll.scrollTop;
           if(this.top>0){
                this.$emit('scrollTop',0)
           }else{
               this.$emit('scrollTop',this.$refs.tlstScroll.scrollTop)
           }
       },
       async infiniteHandler($state){
           this.$emit('onPullingUp',$state)
       },
       touchstartFu(event){
           if(this.refreshing || this.refreshEnd){
               event.preventDefault(); 
               return;
           }
           let touch = event.touches[0];
           this.start = touch.pageY;  // 辅助变量：触摸开始时，相对于文档顶部的Y坐标
       },
       touchmoveFu(event){
           if(this.refreshing || this.refreshEnd || !this.start){
               event.preventDefault(); 
               return;
           }
           this.ismove = true;
           let touch = event.touches[0];
           
            let num = touch.pageY - this.start;
            let y = (num)/2;
            let scrollTop = this.$refs.tlstScroll.scrollTop;
           if(scrollTop<=0){
                if (y < 0) y = 0;
                if (y > this.length) y = this.length;
                let percentage = y / this.length;
                this.rotate = 360 * percentage;
                this.opacity = y / Math.abs(this.initY);
                this.transformStr = `translate(-50%,-50%) rotate(${this.rotate}deg)`;
                this.top = y;
                this.refresh = false;
                if(y>=this.length-50) {
                    this.refresh = true;
                }
            }else{
                this.$emit('scrollTop',scrollTop)

            }
            if(this.top>0){
                this.$emit('scrollTop',0)
                try {
                    event.preventDefault && event.preventDefault(); 
                } catch (error) {
                    
                }
            }
       },
       touchendFu(event){
           this.endFu(event,'istouchend')
       },  
       init(){
            this.initTimeFu = setInterval(()=>{
                    if(this.ismove){
                        this.initTimeFu && clearInterval(this.initTimeFu)
                    }
                    if(this.top<=0){
                        this.initTimeFu && clearInterval(this.initTimeFu)
                        this.setInit();
                        return;
                    }
                    this.top -= 6;
                    this.rotate-=12;
                    this.transformStr = `translate(-50%,-50%) rotate(${this.rotate}deg)`;
                    if(this.top<this.length-50){
                        this.opacity -= 0.2;
                    }
            },20) 
       },
       setInit(){
            this.top = 0;
            this.transformStr = `translate(-50%,-50%) rotate(0deg)`;
            this.opacity = 0;
       },
       async endFu(event,typeStr){
            if(this.refreshing || this.refreshEnd){
               event.preventDefault(); 
               return;
           }
            if(this.istouchend){return}
            if(this.istouchcancel){return}
            this.ismove = false;
           if(this.refresh){
                if(this.top<this.length-50){return}
                this[typeStr] = true;
               this.refreshing = true;
               await new Promise((resolve)=>{
                   this.$emit('onPullingDown',resolve)
               })
                this.refreshEnd = true;
                this.start = "";
                setTimeout(()=>{
                    this.setInit();
                    this.$nextTick(function(){
                        this.refreshEnd = false;
                        this.refreshing = false;
                        this[typeStr] = false;
                    })
                },500)
           }else{
               this.init();
           }
       },
       touchcancelFu(event){
           this.endFu(event,'istouchcancel')
       }    
   },
   activated(){
       this.$refs.tlstScroll.scrollTo({top:Math.max(this.scrollTop - 1, 0),left:0,behavior:"smooth"})
   },
   deactivated(){

   },
   beforeDestroy(){
       if(this.isScrollTop){
           this.$refs.tlstScroll.removeEventListener('scroll',this.scrollFu,false)
       }
       this.$refs.tlstScroll.removeEventListener('touchstart',this.touchstartFu,false)
       this.$refs.tlstScroll.removeEventListener('touchmove',this.touchmoveFu,false)
       this.$refs.tlstScroll.removeEventListener('touchend',this.touchendFu,false)
       this.$refs.tlstScroll.removeEventListener('touchcancel',this.touchcancelFu,false)
   }
}
</script>

