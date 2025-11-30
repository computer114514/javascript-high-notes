# JS中的call、apply、bind方法

改变this的指向

## 一，基本用法与工作原理

### 1，call-“打电话立即执行”

~~~js
//定义一个带有name和招待函数的对象
const person={
    name:'Alice',
    greet:function(greeting,punctuation){
        console.log(`${greeting},我是${this.name}${punctuation}`)
    }
}

//正常调用-this指向person
person.greet('你好','!');//你好，我是Alice！

const newPerson={
    name:'bob'
};

//使用call立刻执行，this指向newPerson
person.greet.call(newPerson,'hi','~');
//hi,我是bob~

//call的模拟实现
Function.prototype.myCall=function(context,...args){
    context=context||window;
    //默认全局对象(应对没有传参的情况)
    const fnSymbol=Symbol();//创建唯一key
    context[fnSymbol]=this;
    //将函数临时创建到context
    const result=context[fnSymbol](...args)
    delete context[fnSymbol]//清理临时属性
    return result;
}；
person.greet.myCall(anotherPerson,'hello','!')
//hello,我是bob!
~~~

[^symbol]: 定义唯一的属性

~~~js
//自动实现call
var target={
    hobby:'eating'
}

function use(name,age){
    console.log(`my name is${name},l am ${age},my hobby is ${this.hobby}`)
}

use.call(target,'alice',18);//alice,18,eating
//手动实现call
Function.prototype.myCall=function(context,...args){
    if(context!=null&&context!==undefined&&typeof context !=='object'&&typeof context!=='function'){
        throw new TypeError("error！")
    }
    //处理context为空的情况
    context=context||window
 
    const newShuXing=Symbol()
    context[newShuXing]=this;
    //this就是Function！
    const result =context[newShuXing](...args);
    delete context[newShuXing];
	return result;
}
use.myCall(target,"alice",'23');

~~~

### 2,apply-"参数打包传递"

~~~Js
//使用apply，参数以数组形式传递
person.greet.apply(anotherPerson,['hey','hi']);
//hey,我是Bob！

//apply工作原理模拟
Function.prototype.myApply=function(context,argsArray){
    context=context||window
    const fnSymbol=Symbol()
    context[fnSymbol]=this;
    const result=context[fnSymbol](...argsArray);
    //...是展开运算符,把数组打散成一个个独立的参数
    //展开数组参数
    delete context[fnSymbol]
    return result
}
person.greet.apply(anotherPerson,['wow','~'])
//wow,我是bob~
~~~

### 3，bind - "预设好稍后执行"

~~~js
//使用bind-返回新函数
const greetBob=person.greet.bind(anotherPerson,'Hello');
greetBob("!!!");//hello,我是bob！！！

//可以分批传参
const greetBobWithHi=person.greet.bind(anotherPerson,'hi','?');
greetBobWithHi();//hi,我是bob？

Function.prototype.myBind=function(context,...bindArgs){
    const self=this;
    return function(...callArgs){
        return self.apply(context,bindArgs.concat(callArgs));
    };
};
//concat合并两个数组
//使用自定义bind
const greetAlice=person.greet.myBind({name:'Alice'},'hey');
greetAlice("!");
//Hey,我是Alice!;
~~~

## 二，实际运用场景

#### 1，**借用**数组**方法**处理**<u>类</u>数组对象**

~~~js
const arrayLike={
0：'apple',
1:'banana',
    length:2
};
//运用apply，传入数组
Array.prototype.push.apply(arrayLike,['orange','grape']);
console.log(arrayLike);//{0：'apple',1:'banana',2:'orange',3:'grape',length:2}

//使用call调用数组方法
console.log(Array.prototype.join.call(arrayLike,[-]))
//"apple-banana-orange-grape"
~~~

[^join]: 以传入的符号转换合并数组，返回一个字符串

#### 2，**事件处理**函数中的this绑定

~~~js
class Button{
    constructor(){
  	this.text='点击我'
 		//运用bind固定this指向           
	this.handleClick=this.handleClick.bind(this);
    }
    handleClick(){
        console.log(`按钮文字：${this.text}`);
    }
}
const btn=new Button();
document.querySelector('button').addEventListener("click",btn.handleClick);
//如果不bind，this指向dom元素
~~~

this.text的this这时已经指向了constructor里的this

bind之后，这个this就永远指向**Button类创造出的实例**了

#### 3，函数科里化

~~~Js
function multiply(a,b,c){
	return a*b*c;
}
const double=multipy.bind(null,2);//固定第一个参数为2
console.log(double(3,4));//24
const triple=multipy.bind(null,3);//固定第一个参数为4
console.log(triple(3,5,6))//90
~~~

## 三，深度原理

### 1,this绑定本质

~~~Js
//普通调用---->this指向window
//方法调用---->this指向方法
//call/bind/apply---->this指向第一个参数
//箭头函数this---->this指向词法作用域的this
~~~

### 2，三兄弟的区别

| 方法  | 参数处理 | 执行时机 | 典型用途               |
| ----- | -------- | -------- | ---------------------- |
| call  | 参数列表 | 立刻     | 明确参数个数           |
| apply | 参数数组 | 立刻     | 参数个数不确定         |
| bind  | 参数列表 | 延后     | 需要固定this或部分参数 |

### 3，性能考量

call通常快于apply（参数处理更简单）

bind有额外开销（要创建新函数）

在热点代码中频繁使用bind可能影响性能

## 四，现代js的替代方案

### 1，箭头函数自动绑定this

~~~Js
class ModernButton{
    constructor(){
        this.text='点击我'；
    }
    //箭头函数会自动绑定this
    handle=()=>{
        console.log(`按钮文字：${this.text}`);
    }
}
~~~

### 2，展开运算符替换apply

~~~js
const nums=[1,2,3];
//以前使用apply
Math.max.apply(null,nums);//3
//现在用展开运算符
Math.max(...nums);//3
~~~

### 3，装饰器提案（stage2）

~~~Js
//未来可能的标准方式
function bind(target,name,descripter){
    const original=descriptor.value;
    descripter.value=original.bind(target);
    return descriptor;
}

class FutureButton{
    @bind
    handleClick(){
        //......
    }
}
~~~

## 五，常见误区和陷阱

#### 1，忽略null/undefined上下文

~~~js
function showName(){
    console.log(this.name);
}

//传入null、undefined，非严格模式下this指向全局
showName.call(null);//可能输出全局name或者undedined

//安全做法：传入空对象
showName.call(Object.create(null))//保证纯净上下文
~~~

#### 2，<u>多次</u>bind<u>无</u>效果

~~~Js
const obj1={name:'Alice'};
const obj2={name:'Bob'};

function greet(){
	console.log(this.name);
}
const bound1=greet.bind(obj1);
const bound2=bound1.bind(obj2);
//无效！this仍然指向obj1
bound2()//ALice
~~~

#### 3,箭头函数无法改变this

~~~Js
const arrowFunc=()=>console.log(this);

arrowFunc.call({name:'Alice'})
//this不会改变，仍然是词法作用域的this
~~~

## 六，最佳实践指南

### 1，方法选择流程图

~~~Js
需要立刻执行吗？--no--bind
|
yes
|
apply,call
参数是数组吗---yes---apply
|
no
|
call
~~~

### 2，性能优化建议

~~~Js
1，缓存bind结果：避免重复bind某一函数
2，优先使用call，(参数确定时)call比apply更高效
3，慎用多层bind:性能问题和难以追踪的bug
4，考虑箭头函数：在类方法是更好的选择
~~~

### 3，代码可读性技巧

~~~Js
someFunc.call(obj,arg1,arg2,arg3)//难理解

const boundFunc=someFunc.bind(obj);
boundFunc(arg1,arg2,arg3);//好写法

const context={详细说明}；
const params=[详细说明]；
someFunc.apply(context,params);
~~~

## 总结

call:打电话，直接说参数（参数列表）

apply:像发邮件，福建打包发送(参数数组)

bind:像设置闹钟，预设好时间再响(延迟执行)