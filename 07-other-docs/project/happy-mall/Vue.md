### 安装vue
```
# 最新稳定版
$ npm install vue
```
### 1、vue声明式渲染
```javascript
let vm = new Vue({
            el: "#app",//绑定元素
            data: {  //封装数据
                name: "张三",
                num: 1
            },
            methods:{  //封装方法
                cancle(){
                    this.num -- ;
                },
                hello(){
                    return "1"
                }
            }
        });
```
### 2、双向绑定,模型变化，视图变化。反之亦然
双向绑定使用v-model
```html
<input type="text" v-model="num">
```
```shell
<h1> {{name}} ,非常帅，有{{num}}个人为他点赞{{hello()}}</h1>
```
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336511-123c9554-c658-4584-a71f-f2a33a42861a.png#align=left&display=inline&height=622&margin=%5Bobject%20Object%5D&originHeight=622&originWidth=937&status=done&style=none&width=937)
### 3、事件处理
v-xx：指令
1、创建vue实例，关联页面的模板，将自己的数据（data）渲染到关联的模板，响应式的
2、指令来简化对dom的一些操作。
3、声明方法来做更复杂的操作。methods里面可以封装方法。
v-on是按钮的单击事件：
```
<button v-on:click="num++">点赞</button>
```
在VUE中el,data和vue的作用:

- el：用来绑定数据；
- data:用来封装数据；
- methods：用来封装方法，并且能够封装多个方法，如何上面封装了cancell和hello方法。

安装“Vue 2 Snippets”，用来做代码提示
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336558-5894d3c5-0be5-4a4e-869c-83371c90bf59.png#align=left&display=inline&height=405&margin=%5Bobject%20Object%5D&originHeight=405&originWidth=732&status=done&style=none&width=732)
为了方便的在浏览器上调试VUE程序，需要安装“[vue-devtools](https://github.com/vuejs/vue-devtools)”，编译后安装到chrome中即可。
详细的使用方法见：[Vue调试神器vue-devtools安装](https://www.jianshu.com/p/63f09651724c)
“v-html”不会对于HTML标签进行转义，而是直接在浏览器上显示data所设置的内容;而“ v-text”会对html标签进行转义
```shell
<div id="app">
        {{msg}}  {{1+1}}  {{hello()}}<br/>
        <span v-html="msg"></span>
        <br/>
        <span v-text="msg"></span>
    </div>
   
    <script src="../node_modules/vue/dist/vue.js"></script>
    <script>
        new Vue({
            el:"#app",
            data:{
                msg:"<h1>Hello</h1>",
                link:"http://www.baidu.com"
            },
            methods:{
                hello(){
                    return "World"
                }
            }
        })
    </script>
```
```
    <script>
        new Vue({
            el:"#app",
            data:{
                msg:"<h1>Hello</h1>"
            },
            methods:({
                hello(){
                    return "World"
                }
            })
        })
    </script>
```
运行结果：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336611-81c8fc8b-662a-4bd7-be1a-0e3f97a51275.png#align=left&display=inline&height=239&margin=%5Bobject%20Object%5D&originHeight=239&originWidth=827&status=done&style=none&width=827)
{{msg}}  :称为差值表达式，它必须要写在Html表达式，可以完成数学运算和方法调用
### 4、v-bind :单向绑定
给html标签的属性绑定
```shell
<!-- 给html标签的属性绑定 -->
    <div id="app"> 
        <a v-bind:href="link">gogogo</a>
        <!-- class,style  {class名：加上？}-->
        <span v-bind:class="{active:isActive,'text-danger':hasError}"
          :style="{color: color1,fontSize: size}">你好</span>
    </div>
    <script src="../node_modules/vue/dist/vue.js"></script>
    <script>
        let vm = new Vue({
            el:"#app",
            data:{
                link: "http://www.baidu.com",
                isActive:true,
                hasError:true,
                color1:'red',
                size:'36px'
            }
        })
    </script>
```
上面所完成的任务就是给a标签绑定一个超链接。并且当“isActive”和“hasError”都是true的时候，将属性动态的绑定到，则绑定该“active”和 "text-danger"class。这样可以动态的调整属性的存在。
而且如果想要实现修改vm的"color1"和“size”， span元素的style也能够随之变化，则可以写作v-bind:style，也可以省略v-bind。
### 5、v-model双向绑定
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <!-- 表单项，自定义组件 -->
    <div id="app">
        精通的语言：
            <input type="checkbox" v-model="language" value="Java"> java<br/>
            <input type="checkbox" v-model="language" value="PHP"> PHP<br/>
            <input type="checkbox" v-model="language" value="Python"> Python<br/>
        选中了 {{language.join(",")}}
    </div>
    
    <script src="../node_modules/vue/dist/vue.js"></script>
    <script>
        let vm = new Vue({
            el:"#app",
            data:{
                language: []
            }
        })
    </script>
</body>
</html>
```
上面完成的功能就是通过“v-model”为输入框绑定多个值，能够实现选中的值，在data的language也在不断的发生着变化，
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336669-930174cb-e1b3-4996-8ba4-ba5340c6ba4c.png#align=left&display=inline&height=469&margin=%5Bobject%20Object%5D&originHeight=469&originWidth=1235&status=done&style=none&width=1235)
如果在控制台上指定vm.language=["Java","PHP"]，则data值也会跟着变化。
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336713-2ed083bf-a664-4dad-8d52-7709411acd0b.png#align=left&display=inline&height=800&margin=%5Bobject%20Object%5D&originHeight=800&originWidth=1401&status=done&style=none&width=1401)
通过“v-model”实现了页面发生了变化，则数据也发生变化，数据发生变化，则页面也发生变化，这样就实现了双向绑定。
数组的连接操作： 选中了 {{language.join(",")}}
### 6、v-on为按钮绑定事件
```shell
<!--事件中直接写js片段-->
        <button v-on:click="num++">点赞</button>
        <!--事件指定一个回调函数，必须是Vue实例中定义的函数-->
        <button @click="cancle">取消</button>
```
上面是为两个按钮绑定了单击事件，其中一个对于num进行自增，另外一个自减。
v-on:click也可以写作[_@_click ](https://www.yuque.com/click)
事件的冒泡：
```html
<!-- 事件修饰符 -->
        <div style="border: 1px solid red;padding: 20px;" v-on:click="hello">
            大div
            <div style="border: 1px solid blue;padding: 20px;" @click="hello">
                小div 

                <a href="http://www.baidu.com" @click.prevent="hello">去百度</a>
            </div>
        </div>
```
上面的这两个嵌套div中，如果点击了内层的div，则外层的div也会被触发；这种问题可以事件修饰符来完成：
```html
<!-- 事件修饰符 -->
        <div style="border: 1px solid red;padding: 20px;" v-on:click.once="hello">
            大div
            <div style="border: 1px solid blue;padding: 20px;" @click.stop="hello">
                小div 

                <a href="http://www.baidu.com" @click.prevent.stop="hello">去百度</a>
                <!--这里禁止了超链接的点击跳转操作，并且只会触发当前对象的操作-->
            </div>
        </div>
```
关于事件修饰符：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336764-a1fa6d7a-157c-4d53-ae4c-0504cbfbdb9a.png#align=left&display=inline&height=402&margin=%5Bobject%20Object%5D&originHeight=402&originWidth=1084&status=done&style=none&width=1084)
按键修饰符：
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336822-515e5adb-dc25-49f3-8e35-3b36b3e9452c.png#align=left&display=inline&height=665&margin=%5Bobject%20Object%5D&originHeight=665&originWidth=1127&status=done&style=none&width=1127)
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336865-2da6ad4d-8295-430e-8866-bdcd1c71e63c.png#align=left&display=inline&height=204&margin=%5Bobject%20Object%5D&originHeight=204&originWidth=1164&status=done&style=none&width=1164)
### 7、v-for遍历循环
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div id="app">
        <ul>
            <li v-for="(user,index) in users" :key="user.name" v-if="user.gender == '女'">
                <!-- 1、显示user信息：v-for="item in items" -->
               当前索引：{{index}} ==> {{user.name}}  ==>   {{user.gender}} ==>{{user.age}} <br>
                <!-- 2、获取数组下标：v-for="(item,index) in items" -->
                <!-- 3、遍历对象：
                        v-for="value in object"
                        v-for="(value,key) in object"
                        v-for="(value,key,index) in object" 
                -->
                对象信息：
                <span v-for="(v,k,i) in user">{{k}}=={{v}}=={{i}}；</span>
                <!-- 4、遍历的时候都加上:key来区分不同数据，提高vue渲染效率 -->
            </li>
            
        </ul>
        <ul>
            <li v-for="(num,index) in nums" :key="index"></li>
        </ul>
    </div>
    <script src="../node_modules/vue/dist/vue.js"></script>
    <script>         
        let app = new Vue({
            el: "#app",
            data: {
                users: [{ name: '柳岩', gender: '女', age: 21 },
                { name: '张三', gender: '男', age: 18 },
                { name: '范冰冰', gender: '女', age: 24 },
                { name: '刘亦菲', gender: '女', age: 18 },
                { name: '古力娜扎', gender: '女', age: 25 }],
                nums: [1,2,3,4,4]
            },
        })
    </script>
</body>
</html>
```
4、遍历的时候都加上:key来区分不同数据，提高vue渲染效率



   <!-- 
        v-if，顾名思义，条件判断。当得到结果为true时，所在的元素才会被渲染。
        v-show，当得到结果为true时，所在的元素才会被显示。 
    -->
    <div id="app">
        <button v-on:click="show = !show">点我呀</button>
        <!-- 1、使用v-if显示 -->
        <h1 v-if="show">if=看到我....</h1>
        <!-- 2、使用v-show显示 -->
        <h1 v-show="show">show=看到我</h1>
    </div>
    <script>
        let app = new Vue({
            el: "#app",
            data: {
                show: true
            }
        })
    </script>
<script>
        //watch可以让我们监控一个值的变化。从而做出相应的反应。
        new Vue({
            el: "#app",
            data: {
                xyjPrice: 99.98,
                shzPrice: 98.00,
                xyjNum: 1,
                shzNum: 1,
                msg: ""
            },
            computed: {
                totalPrice(){
                    return this.xyjPrice*this.xyjNum + this.shzPrice*this.shzNum
                }
            },
            watch: {
                xyjNum(newVal,oldVal){
                    if(newVal>=3){
                        this.msg = "库存超出限制";
                        this.xyjNum = 3
                    }else{
                        this.msg = "";
                    }
                }
            },
        })
    </script>
### 过滤器
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <!-- 过滤器常用来处理文本格式化的操作。过滤器可以用在两个地方：双花括号插值和 v-bind 表达式 -->
    <div id="app">
        <ul>
            <li v-for="user in userList">
                {{user.id}} ==> {{user.name}} ==> {{user.gender == 1?"男":"女"}} ==>
                {{user.gender | genderFilter}} ==> {{user.gender | gFilter}}
                <!-- 这里的"|"表示的管道，将user.gender的值交给genderFilter -->
            </li>
        </ul>
    </div>
    <script src="../node_modules/vue/dist/vue.js"></script>
    <script>
        // 全局过滤器 
        Vue.filter("gFilter", function (val) {
            if (val == 1) {
                return "男~~~";
            } else {
                return "女~~~";
            }
        })
        let vm = new Vue({
            el: "#app",
            data: {
                userList: [
                    { id: 1, name: 'jacky', gender: 1 },
                    { id: 2, name: 'peter', gender: 0 }
                ]
            },
            filters: {
                //// filters 定义局部过滤器，只可以在当前vue实例中使用
                genderFilter(val) {
                    if (val == 1) {
                        return "男";
                    } else {
                        return "女";
                    }
                }
            }
        })
    </script>
</body>
</html>
```
### 组件化
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div id="app">
        <button v-on:click="count++">我被点击了 {{count}} 次</button>
        <counter></counter>
        <counter></counter>
        <counter></counter>
        <counter></counter>
        <counter></counter>
        <!-- 使用所定义的组件button-counter -->
        <button-counter></button-counter>
    </div>
    <script src="../node_modules/vue/dist/vue.js"></script>
    <script>
        //1、全局声明注册一个组件
        Vue.component("counter", {
            template: `<button v-on:click="count++">我被点击了 {{count}} 次</button>`,
            data() {
                return {
                    count: 1
                }
            }
        });
        //2、局部声明一个组件
        const buttonCounter = {
            template: `<button v-on:click="count++">我被点击了 {{count}} 次~~~</button>`,
            data() {
                return {
                    count: 1
                }
            }
        };
        new Vue({
            el: "#app",
            data: {
                count: 1
            },
            components: {
                //声明所定义的局部组件
                'button-counter': buttonCounter
            }
        })
    </script>
</body>
</html>
```
![](https://cdn.nlark.com/yuque/0/2020/png/512093/1591341336925-12a20bb8-3166-4c02-9a0e-b27af23c4cb2.png#align=left&display=inline&height=225&margin=%5Bobject%20Object%5D&originHeight=225&originWidth=911&status=done&style=none&width=911)
### 生命周期钩子函数
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div id="app">
        <span id="num">{{num}}</span>
        <button @click="num++">赞！</button>
        <h2>{{name}}，有{{num}}个人点赞</h2>
    </div>
    <script src="../node_modules/vue/dist/vue.js"></script>
    
    <script>
        let app = new Vue({
            el: "#app",
            data: {
                name: "张三",
                num: 100
            },
            methods: {
                show() {
                    return this.name;
                },
                add() {
                    this.num++;
                }
            },
            beforeCreate() {
                console.log("=========beforeCreate=============");
                console.log("数据模型未加载：" + this.name, this.num);
                console.log("方法未加载：" + this.show());
                console.log("html模板未加载：" + document.getElementById("num"));
            },
            created: function () {
                console.log("=========created=============");
                console.log("数据模型已加载：" + this.name, this.num);
                console.log("方法已加载：" + this.show());
                console.log("html模板已加载：" + document.getElementById("num"));
                console.log("html模板未渲染：" + document.getElementById("num").innerText);
            },
            beforeMount() {
                console.log("=========beforeMount=============");
                console.log("html模板未渲染：" + document.getElementById("num").innerText);
            },
            mounted() {
                console.log("=========mounted=============");
                console.log("html模板已渲染：" + document.getElementById("num").innerText);
            },
            beforeUpdate() {
                console.log("=========beforeUpdate=============");
                console.log("数据模型已更新：" + this.num);
                console.log("html模板未更新：" + document.getElementById("num").innerText);
            },
            updated() {
                console.log("=========updated=============");
                console.log("数据模型已更新：" + this.num);
                console.log("html模板已更新：" + document.getElementById("num").innerText);
            }
        });
    </script>
</body>
</html>
```
## element ui
官网： [https://element.eleme.cn/#/zh-CN/component/installation](https://element.eleme.cn/#/zh-CN/component/installation)
安装
```shell
npm i element-ui -S
```
在 main.js 中写入以下内容：
```
import ElementUI  from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css';
Vue.use(ElementUI);
```
