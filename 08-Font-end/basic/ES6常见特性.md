# 简介
HTML 定义了网页的内容；CSS 描述了网页的布局；JavaScript网页的行为。
ECMAScript是浏览器脚本语言的规范。ES6，是于2015.06发布的一个JavaScript 版本标准。

# ES6新特性
## 1、let声明变量
（1）var声明变量往往会越域；let声明的变量有严格的局部作用域    
```javascript
    {        
   	var a=1;        
    let b=2;    
    }    
    console.log(a);    // 1
    console.log(b);    // Uncaught ReferenceError: b is not defined
```
（2） var可以声明多次；let只能声明一次     
```javascript   
        var m = 1;            
        var m = 2;            
        let n = 1;            
        let n = 2;                
        console.log(m); // 2        
        console.log(n); // Identifier 'n' has already been declared
```
（3）var存在变量提升；let不存在变量提升
* 变量提升：函数声明和变量声明总是会被解释器悄悄地被"提升"到方法体的最顶部。
```javascript
        console.log(x);        
        var x = 0;      // undefined（变量提升只是声明（var x）提升，而初始值(x=7)是不能提升的。）
        console.log(y);        
        let y = 1;      //  Cannot access 'y' before initialization
```
* 以上实例相当于以下代码：
```javascript
        var x;
        console.log(x);        

        console.log(y);        
        let y = 1;
```
## 2、const声明常量（只读变量）
声明必须初始化，声明后不允许改变。

```javascript
        const x = 1;        
        x = 1;  // Uncaught TypeError: Assignment to constant variable.
```
## 3、解构表达式
（1）数组解构

```javascript
        let arr = [1, 2, 3];
        //let a = arr[0];        
        //let b = arr[1];        
        //let c = arr[2];
       
        let [a, b, c] = arr; // 数组解构        
        console.log(a, b, c);
```
（2）对象解构

```javascript
        const person = {            
            name: "jack",            
            age: 21,            
            language: ['Java', 'CSS', 'Js']        
        }
        // const name = person.name;        
        // const age = person.age;        
        // const language = person.language;                
        
        const { name, age, language } = person; // 对象解构
        console.log(name, age, language)
```
## 4、字符串扩展
（1）新的API
```javascript
        let str="hello.vue"
        console.log(str.startsWith("hello"));// true        
        console.log(str.endsWith(".vue"));// true        
        console.log(str.includes("e"));// true        
        console.log(str.includes("hello"));// true
```
（2）字符串模板
* 多行字符串    
```javascript
        let str = `<head>            
        	<meta charset="UTF-8">`
        console.log(str);
```
* 字符串插入变量和表达式。变量写在${ }中，${}中可以加入JavaScript表达式。

```javascript
        const person = {            
            name: "jack",            
            age: 21,            
            language: ['Java', 'CSS', 'Js']        
        }
        const { name, age, language } = person;
        let info = `我是${name},今年${age}了`;
        console.log(info); // 我是jack,今年21了
```
* 字符串调用函数

```javascript
        const person = {            
            name: "jack",            
            age: 21,            
            language: ['Java', 'CSS', 'Js']        
        }
        const { name, age, language } = person;
        function fun(){
            return "这是一个函数。";        
        }
        let info = `我是${name},今年${age}了,我想说：${fun()}`;
        console.log(info); //我是jack,今年21了,我想说：这是一个函数。
```
## 5、函数优化
（1）函数参数默认值

```javascript
        // SE6之前，无法给函数一个参数设置默认值，只能采用变通写法：        
        function add(a, b) {            
            // 判断b是否为空，为空就给默认值1            
            b = b || 1;            
            return a + b;        
        }        
        // 传一个参数        
        console.log(add(10)); // 11
        
        // SE6，直接给参数写上默认值，没穿就会自动使用默认值        
        function add2(a, b = 1) {            
            return a + b;        
            }        
        console.log(add2(20)); // 21
```
（2）不定参数

```javascript
        function fun(...values) {            
            console.log(values.length)        
        }        
        fun(1, 2)// 2        
        fun(1, 2, 3, 4)// 4
```
（3）箭头函数

```javascript
        // 以前声明方法        
        var print = function (obj) {            
            console.log(obj);        
        }
        // SE6，箭头函数传一个参数        
        var print = obj => console.log(obj);        
        
        var sum = function (a, b) {            
            return a + b;        
        }
        print("hello");// hello
        // SE6，箭头函数传多个参数        
        var sum2 = (a, b) => a + b;
        console.log(sum2(11, 12));// 33
        var sum3 = (a, b) => {            
            c = a + b;            
            return a + c;        
        }        
        console.log(sum3(10, 20));// 40
        
        const person = {            
            name: "jack",            
            age: 21,            
            language: ['java', 'js', 'css']        
        }
        // 之前写法        
        function hello(person) {            
            console.log("hello," + person.name)        
        }        
        // 箭头函数        
        var hello2 = (obj) => console.log("hello," + obj.name);        
        hello2(person); // hello,jack       
        // 箭头函数+解构表达式        
        var hello2 = ({name}) => console.log("hello," + name);       
        hello2(person); // hello,jack
```
## 6、对象优化
（1）新增API
```javascript
        const person = {            
            name: "jack",
            age: 21,            
            language: ['java', 'js', 'css']        
        }        
        // 获取对象中所有属性        
        console.log(Object.keys(person));//["name","age","language"]        
        // 获取对象中所有属性值        
        console.log(Object.values(person));//["jack",21,Array(3)]        
        // 获取K-Value        
        console.log(Object.entries(person));//[Array(2),Array(2),Array(2)]
        
        const target = { a: 1 };        
        const source1 = { b: 2 };        
        const source2 = { c: 3 };        
        // 第一个参数式目标对象，后面的对象都是源对象        
        Object.assign(target, source1, source2);       
        console.log(target);// {a: 1, b: 2, c: 3}
```
（2）声明对象简写
==属性名与属性值变量名相同时，可以省略。==
```javascript
        const age = 23;        
        const name = "张三";        
        //const person1 = { age:age, name:name };        
        const person1 = { age, name };        
        console.log(person1);// { 23, 张三 }
```
（3）对象函数属性简写
![](https://img-blog.csdnimg.cn/20200731155515701.png)
（4）拓展运算符（...）用于取出参数对象所有可遍历属性然后拷贝到当前对象。
![](https://img-blog.csdnimg.cn/20200731160343354.png)

## 7、数组新增方法：map和reduce
![](https://img-blog.csdnimg.cn/20200803092904706.png)
## 8、Promise
![](https://img-blog.csdnimg.cn/20200803103925108.png)

## 9、模块化
* hello.js

```javascript
// export const util = {
//     sum(a, b) {
//         return a + b;
//     }
// }
export default {    
    sum(a, b) {        
        return a + b;    
    }
}
// export {util}
//`export`不仅可以导出对象，一切JS变量都可以导出。比如：基本类型变量、函数、数组、对象。
```
* user.js
```javascript
var name = "jack"
var age = 21
function add(a,b){    
    return a + b;
}
export {name,age,add}
```
* main.js调用
```javascript
import abc from "./hello.js"
import {name,add} from "./user.js"
abc.sum(1,2);
console.log(name);add(1,3);
```