# 深入之call、apply的模拟实现

## 1，CALL

> call()方法再一个指定的this和若干指定参数的前提下调用某个函数、方法

举个例子：

~~~js
var=foo{
    value:1
}
function bar(){
    console.log(this.value)
    //这是默认绑定
    //this指的是window里边儿的value！
}
bar.call(foo);//1
~~~

注意两点：
1，call**改变this的指向**，指向foo

2，bar函数执行，foo的value被打印出来了

## 2，模拟实现第一步

模拟实现欸~是指模拟实现这个call函数啦

——把foo改造一下

~~~Js
var foo={
    value:1;
    bar:function(){
        console.log(this.value)
    }
};

foo.bar();//1
//foo是调用者，这是隐式绑定
~~~

可是。。。。。。。foo莫名多了一个value，这时候需要delete掉它了

##### delete方法：

1，将函数设为**对象的属性**

2，**执行**该函数

3，**删除**该函数

~~~js
//1，
foo.fn=bar
//2,
foo.fn()
//3,
delete foo.fn
~~~

fn是属性名，取什么无所谓，因为到时候总会删掉的。

so，总结call2函数：

~~~js
//第一版
Function.prototype.call2=function(context，...args){
    //1，检查调用者是不是函数
    if(typeof this!=='function'){
        throw new Type('Function.prototype.call called on non-function')
    }
    
    //2，处理context，null/undefiend的话就指向全局对象
    context=context||globalThis
    //浏览器中为window，Node中为global
    
    //3，为context添加唯一属性，避免覆盖原有方法
    const fnKey=Symbol('tempFn');
    context[fnKey]=this;
    //this指向调用myCall的函数
    
    
    
    //首先要获取调用call的函数，用this可以获取
    context.fn=this;
    //this应该是function
    context.fn();
    //此时执行function函数this指向了value！
    delete context.fn;
}

//测试一下
var foo={
    value:1
};
function bar(){
    console.log(this.value)
}
bar.call2(foo);//1
~~~

先声明一个对象foo，声明bar函数；

call2模拟把bar的this指向了foo；

## 2，模拟实现第二步

最一开始也讲了，call函数还能给**指定参数**执行函数

~~~js
var foo={
    value：1
}；
function bar（name，age）{
    console.log(name)
    console.log(age)
    console.log(this.value)
}
bar.call(foo,'kvein',18)
//kevin
//18
//1
~~~

注意：传入的参数并不确定，怎么办？

从arguments对象里取值，取出第二个到最后一个参数，然后放到一个数组里。