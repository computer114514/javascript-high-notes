## <u>**？谈谈你对this的理解**</u>

# T · H · I · S

## 一，定义 

函数的this关键字在js中的表现略有不同，此外，在严格模式和非严格模式之间也有差别

函数的调用方式决定了this的值（执行上下文创建时确定）

this是函数运行时自动生成的一个内部对象，只能在函数内部使用，总是指向**调用它的对象**

~~~js
function baz(){
    //当前调用栈是：baz
    //因此，当前调用位置是全局作用域
    console.log("baz");
    bar();//bar的调用位置
}
function bar(){
    //baz-bar
    //因此，当前调用位置在baz中
    console.log("bar")
    foo();//foo的调用位置
}
function foo(){
	//baz-bar-foo
    //当前调用位置在bar中
    console.log("foo")
}
baz();//baz调用位置
~~~

函数执行的时候，this不可被修改

## 二，绑定规则

this的值不同取决于不同场合

>默认绑定
>
>隐式绑定
>
>new绑定
>
>显示绑定

### 默认绑定

全局定义函数person，使用this关键字

~~~js
var name="jenny"
function person(){
    return this.name
}
console.log(person())//jenny
~~~

调用的对象位于window，所以this指向window

Attention：严格模式下，this指向undefined，非window

只有**非严格**模式下，默认绑定才能绑定到全局对象

### 隐式绑定

函数被某个对象调用，this就指向这个**上级对象**

~~~js
function test(){
    console.log(this.x)
}
var obj={}
obj.x=1
obj.m=test
obj.m()
//打印出1
~~~

被最外层对象调用，但是this还是指向**最近一级**

~~~Js
var o={
    a:10;
    b:{
    fn:function(){
    console.log(this.a)
		}
	}
}
o.b.fn()
//打印了b中的a，未定义啊，undefined
~~~

😶‍🌫️!Attention：fn()一定要执行，不然this指向window

### new绑定

new一个对象，这时this指向这个实例对象

~~~js
function test(){
   this.x=1
    //调用者的x设置为1；
}
var obj=new test()
obj.x//1
//这说明，obj和this等价，说明this指向了obj
~~~

new过程遇到return一个对象，this指向return的这个对象

~~~js
function fn(){
	this.user="xxx";
    return{}
}
var a= new fn();
console.log(a.user)//undefined
//有return存在，this指向return的这个对象
~~~

返回一个简单类型（非对象）,this还是指向实例

~~~js
function fn(){
    this.user='xxx',
        return 1
}
var a=new fn();
console.log(a.user)//xxx
~~~

### 显式修改

apply(),call(),bind()是函数的一个方法，作用是改变函数的调用对象。

~~~js
var x=0
function test(){
    console.log(this.x)
}
var obj={}
obj.x=1
obj.m=test
obj.m.apply(obj)//1
//apply将m的this改变为了obj

~~~

## 三，箭头函数

书写后就能确定this的指向

this继承**外层**最近的非箭头函数作用于的this(上下文)，且一旦确定就无法改变（静态绑定）

attention:箭头函数不适合作为对象的方法

## 四，优先级

> 隐式绑定<显示绑定

> new绑定>隐式绑定

> new绑定>显示绑定

<u>能否举例子说明优先级问题？</u>

~~~Js
function foo(sth){
    this.a=sth
}
var obj1={
    foo:foo
    //定义foo方法
}
let obj2={}

obj1.foo(2)
console.log(obj1.a)//2
//这是隐式绑定，调用
obj1.foo.call(obj2,3)
console.log(obj2.a)//3
//这是显式绑定，手动call改变

let example =new obj2.foo(4)
console.log(obj2.a)//3
console.log(example.a)//4
//这就是new绑定>隐式绑定啊

let bar=obj1.foo.bind(obj2)
bar(2)
console.log(obj2.a)//2

let baz=new bar(3)
console.log(obj1.a)//2
console.log(baz.a)//3
~~~

所以->绑定this的时候

**<u>new>显式>隐式>模式</u>**

这样的优先级