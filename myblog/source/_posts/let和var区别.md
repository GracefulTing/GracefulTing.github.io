## var和let区别

1. let和const不存在变量提升机制，但是在词法解析阶段也可以得知变量是否是私有变量；
2. var允许重复声明，而let不允许；
3. let能解决typeof检测时出现的暂时性死区问题（let比var更严谨）；
4. let创建的全局变量没有给window设置对应的属性；
5. let会产生块级作用域；
6. ....


```javascript
console.log(a) //undefined
var a = 12;
function fn(){
	console.log(a)   //undefined
	var a = 13;
}
fn()
console.log(a)   //12
//输出结果：undefined  undefined  12
```

```javascript
console.log(a)  // undefined
var a = 12;
function fn(){
	console.log(a) //12
	a = 13
}
fn()
console.log(a)  //13
//输出结果： undefined   12   13
```

```javascript
console.log(a)  //报错 Uncaught ReferenceError（引用错误）: a is not defined
a = 12; 
function fn(){
	console.log(a)
	a = 13
}
fn()
console.log(a)

/* 去掉第一行后 */
a = 12;   //在全局作用域下，带var/function声明的全局变量相当于给window设置了对应的属性(既是全局变量也是属性)，不带var声明的只是给window设置了对应的属性，不是全局变量(eg:a=12)。如果使用的是let/const声明的，只是全局变量，没有给window设置对应的属性(eg:let a=12;)。
function fn(){
	console.log(a)   //12
	a = 13
}
fn()
console.log(a)   //13
// 如果去掉第一行console后输出结果： 12  13
```

```javascript
console.log(a)  //报错 Uncaught ReferenceError: Cannot access 'a' before initialization
let a = 12;
function fn(){
	console.log(a)
	let a = 13;
}
fn()
console.log(a)

/*  去掉第一行依然报错  */
let a = 12;
function fn(){
    //形参赋值 & 变量提升 & 词法解析
    //报错原因：词法解析时已经知道在当前私有栈中有let a,此时出现的a都是私有的
    //在当前作用域下(全局、私有、块作用域)，如果创建变量使用的是let/const等，一定不能在创建代码的前面使用这些变量，否则会报错Uncaught ReferenceError: Cannot access 'a' before initialization
	console.log(a)    //报错 Uncaught ReferenceError: Cannot access 'a' before initialization
	let a = 13;
}
fn()
console.log(a)


let a = 12;
function fn(){
	console.log(a)    //12
}
fn()
console.log(a)  //12
```



let所在大括号是一个块作用域（私有作用域）

```javascript
if(1===1){
	var a = 12;   //没有块作用域，全局
    let b = 13;   //有块作用域，私有
}
console.log(a)   //12
console.log(b)   //报错 Uncaught ReferenceError: b is not defined
```



```javascript
var foo=1;
function bar(){
    //不管条件是否成立都要进行变量提升
    if(!foo){//!undefined->true
        var foo=10;
    }
    console.log(foo);   //10
}
bar();


let foo=1;
function bar(){
    if(!foo){//!1->false
        let foo=10;
    }
    console.log(foo);   //1
}
bar();

let foo=1;
function bar(){
    if(foo){//1->true
        let foo=10;
        console.log(foo);  //10
    }
    console.log(foo);   //1
}
bar();
```

```javascript
let n=12;
~function(){
    if(1){
        let n=13;
    }
    console.log(n);  //12
}();


let n=12;
~function(){
    if(1){
        n=13;
    }
    console.log(n);  //13
}();
console.log(n); //13


let n=12;
~function(){
    let n=0;
    if(1){
        n=13;
    }
    console.log(n);  //13
}();
console.log(n); //12
```



不管条件是否成立，都要进行变量提升。

```javascript
//var a;全局声明a相当于window.a  
if(!("a" in window)){//true
    var a = 1;
}
console.log(a);  //undefined
```



```javascript
/*
	arguments时函数内置的实参集合，箭头函数中没有arguments，不管是否定义了形参，也不管传递了多少实参，arguments中包含所有传递的实参信息（类数组集合）
*/
var a = 4;
function b(x,y,a){
    //在js非严格模式下，arguments和形参存在映射关系（一个改都会跟着改）
    //console.log(arguments);   //{0:1,1:2,2:3,length:3...}
    console.log(a); //3
    arguments[2] = 10;//把传递的第三个实参修改为10，形参也跟着改为10
    console.log(a);//10
}
a = b(1,2,3);  //a=b的执行结果，无return 也就是a=undefined
console.log(a);//undefined


//严格模式下，arguments和形参的映射机制就切断了
"use strict";
function b(x,y,a){
    arguments[2] = 10;
    console.log(a);//3
}
b(1,2,3); 
```



