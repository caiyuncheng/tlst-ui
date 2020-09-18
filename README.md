# tlst-ui

## 安装

可以使用下列命令安装这个包：

``` bash
npm install tlst-ui
```

## 使用

### script
```js
import { tlstScroll } from 'tlst-ui';
components:{
    tlstScroll
},
methods: {
    onPullingDown(next){
        ...逻辑代码
        next && next();
    }
}
```

### template
```js
<tlstScroll @onPullingDown="onPullingDown">
    .....页面代码
</tlstScroll>
```

##### 上拉加载更多是采用其他库
[vue-infinite-loading](https://github.com/PeachScript/vue-infinite-loading)

### 示例

2 个示例代码快速了解如何使用 Scroll 组件。

+ **1. 基本使用 - Default**   
   ```js
      <template>
            <tlstScroll @onPullingDown="onPullingDown">
                  .....内容部分
            </tlstScroll>
      </template>
      <script>
         import { tlstScroll } from 'tlst-ui';
         export default {
            data(){
               return{
               }
            },
            components:{
               tlstScroll
            },
            methods: {
               onPullingDown(next){
                  setTimeout(()=>{
                        next && next();
                        this.$createToast({
                           time: 1500,
                           txt: '延时2秒结束下拉',
                           mask: true,
                        }).show();
                  },2000)
               }
            }
         }
      </script>
   ```

+ **2. 下拉刷新，上拉加载更多 - All**

   ```js
      <template>
         <tlstScroll @onPullingUp="onPullingUp" :isMore="true" @onPullingDown="onPullingDown">
            .....内容部分
         </tlstScroll>
      </template>
      <script>
         import { tlstScroll } from 'tlst-ui';
         export default {
            data(){
               return{
                  maxNum:3,
                  newNum:0,
                  isajax:false,
                  infinite:"",
               }
            },
            components:{
               tlstScroll
            },
            methods: {
               onPullingDown(next){
                  setTimeout(()=>{
                        next && next();
                        this.newNum = 0;
                        this.infinite.reset();//重置
                  },2000)
               },
               async onPullingUp(obj){
                     this.newNum++;
                     if(!this.infinite){
                        this.infinite = obj;
                     }
                     if(this.newNum>this.maxNum){
                        obj.complete();//全部完成
                        return;
                     }
                     if(this.isajax)return;
                     this.isajax = true;
                     await new Promise((resolve)=>{
                        setTimeout(()=>{
                           resolve(1)
                        },2000)
                     });
                     this.isajax = false;
                     obj && obj.loaded && obj.loaded(); //结束这次
               }
            }
         }
      </script>
   ```

---

### Props 配置

| 参数          | 说明                | 类型    |  可选值          | 默认值    |
| ------------- |:------------------:|   -----:|           -----:|     -----:|
| isScrollTop   | 是否需要实时滚动高度 | boolean |  true/false     |      false|
| isMore        | 是否需要上拉加载更多 | boolean |  true/false     |     false |

### 插槽

|  名字  |  说明  |  作用域参数  |
|------  | -----:|     -----    |
|default | 内容体 |       -     |

### 事件

| 事件名         | 说明         | 参数                                 |
|         ----  | -----        |-----                                 | 
| onPullingDown | 下拉刷新      | next 返回回调函数，调用即结束下拉刷新   |
| onPullingUp   | 上拉加载更多  | Object   返回一个对象 `可以用变量保存起来` <br><br>  Object.loaded()  已完成本次上拉加载更多 <br><br> Object.complete()  结束上拉加载更多 <br><br> Object.reset()  重置上拉加载更多|

