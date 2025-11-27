# js深入之类数组对象与arguments

### 类数组对象

就是拥有长度length属性和若干索引属性的对象

~~~js
var array=["name","age","sex"]

var arraylike={
   0:"name",
    1:"age",
    2:"sex",
    length:3
}
~~~

>读写方面区别

~~~Js
console.log(array[0])//name
console.log(arraylike[0])//name

array[0]="new name"
arraylike[0]="new name"
~~~

>长度

~~~Js
console.log(array.length)//3
console.log(arraylike.length)//3
~~~

>遍历

~~~js

~~~

以上操作都可以实现.

但是呢，数组还有很多方法：push、splice等等

但是这样类数组就会报错了：arraylike.push is not a function

### 调用数组方法

那怎么运用这个方法呢？

----运用function.call方法间接调用

~~~js
var arraylike ={0:"name",1:"age",2:"sex",length:3}
//1,slice方法
Array.prototype.slice.call(arrayLike,0);//["name","age","sex"]
//slice可以做到类数组转数组

//返回以&分割的字符串？
Array.prototype.join.call(arrayLike,"&");//name&age&sex

//map生成新数组，每个元素执行item.toUpperCase操作，返回到新数组
Array.prototype.map.call(arrayLike,function(){
	return item.toUpperCase
}
//["NAME","AGE","SEX"]
~~~

### 类数组转对象

啊，上面的slice就是一个数组转对象的方法，还有三个哦，从从帮哦

~~~js
var arrayLike={
    0:'name',
    1:'age',
    2:'sex',
    length:3
}
//1,slice
Array.prototype.slice.call(arrayLike,0)

//2,splice
Array.prototype.splice.call(arrayLike,0)

//3,ES6 Array.from
Array.from(arrayLike);

//4,apply
Array.prototype.concat.apply([],arrayLike)
~~~

so，类数组有啥用啊？

在客户端js中，一些DOM方法（document.getElementsByTagName()等）就返回类数组对象

引出Arguments对象，这就是个类数组对象

### Arguments对象

arguments对象**只**定义在函数体上，包括了函数的参数和其他属性。

函数体中，arguments指代Arguments对象

~~~Js
function foo(name,age,sex){
    console.log(arguments);
}
foo("name","age","sex")


//打印出来的东西
arguments[3]
    0:"name"
    1："age"
    2:"sex"
    callee:foo(name,age,sex)
	//调用函数本身（已经被弃用）
	length:3
	Symbol(Symbol.literator):values()
	__proto__:Object
~~~

哎哟我的天啊，这打印出来了个什么东西？

### length属性

表示实参的长度

~~~js
function foo(a,b,c){
    console.log(arguments.length)//实参的长度为1
}
console.log(foo.length)//形参的长度为3
foo(1)
~~~

### callee属性（已弃用）

Arguments对象的callee属性，通过它可以调用函数本身。

~~~js
var data=[]

for(var i=0;i<3;i++){
    (data[i]=function(){
        console.log(arguments.callee.i)
    }).i=i;
}
data[0]();//0
data[1]();//1
data[2]();//2
~~~

arguments和对应参数的绑定

~~~js
function foo(name,age,sex,hobbit){
    console.log(name,arguments[0]);//name name
    //改变形参
	name='new name'
    console.log(name,arguments[0])//new name,new name
    //wow，会绑定✌，name和arguments中的name
    
    //改变arguments
    arguments[1]="new age";
    
    console.log(age,arguments[1]);//new age,new age
    
    //测试没有传入的会不会绑定
    console.log(sex)//undefined
    sex='new sex'
    console.log(sex,argumenys[2]);//new sex undefiend
    
    arguments[3]="new hobbit";
    
    console.log(hobbit,argument[3])//undefiend new hobbit
}
foo('name','age')
~~~

传入的参数，实参和arguments的值会共享，反之，则不会共享

*我们的严格模式下，这两个就不会共享了

### 传递参数

将参数从一个函数传递到另一个函数

~~~Js
//使用apply将foo的参数传递给bar
function foo(){
    bar.apply(this,arguments);
    //目前还不会apply，所以不理解
}
function bar(a,b,c){
    console.log(a,b,c)
}
foo(1,2,3)
~~~

### 强大的ES6

使用ES6的...运算符，我们可以轻松转成数组。

~~~Js
function func(...arguments){
console.log(arguments);//[1,2,3]
}
func(1,2,3);
~~~

### 应用

arguments的应用很多，后面jQuery的extend实现什么的，都有这个东西的身影。

1，参数不定长

2，函数柯里化

3，递归调用

4，函数重载



