## var & let 的区别

> **变量提升**：当浏览器开辟出供代码执行的栈内存后，代码并没有自上而下立即执行，而是继续做了一些事情：把当前作用域中所有带var/function关键字的进行提前的声明和定义，即变量提升机制。
>
> - 带var的只是提前声明：如果只声明不赋值，默认值为undefined；
> - 带function的不仅声明还定义：准备来说就是创建函数并且让函数与开辟的堆内存地址关联；

1. **let和const不存在变量提升机制，但是在词法解析阶段也可以得知变量是否为私有变量；**

   6种创建变量的方式中，var和function存在变量提升，而let/const/class/import不存在这个机制。

   ```javascript
   console.log(a)   //输出：undefined
   var a = 1;
   //因为变量提升阶段声明了a只是未赋值，赋值前输出a则为undefined
   ```

   ```javascript
   console.log(a)  //报错 Uncaught ReferenceError: Cannot access 'a' before initialization
   let a = 1;
   //let不存在变量提升，在初始化前不能使用
   ```

   ```javascript
   console.log(a); //报错  Uncaught ReferenceError: a is not defined
   ```

2. **var允许重复声明，而let不允许；**

   > 相同作用域（执行上下文）中：
   >
   > - 如果使用var/function声明变量，且重复声明，是不会有影响的（再声明第一次之后就不再重复声明了）
   > - 如果使用let/const就会报错，因为浏览器会校验当前作用域中是否已经存在该变量，如果已经存在，再次声明就会报错

   ```javascript
   console.log(a)
   let a = 1;
   console.log(a)
   let a = 2;     //报错 Uncaught SyntaxError: Identifier 'a' has already been declared
   console.log(a)
   /*
   *  在浏览器开辟占内存供代码自上而下执行前，不仅有变量提升，还有很多别的操作：形参赋值，此法解析等
   *  词法解析(词法检测):检测当前将要执行的代码是否会出现‘语法错误(SyntaxError)’,如果出现错误，代码将不会执行,第一行也不会执行
   */
   ```

   ```javascript
   //重复声明：不管之前通过什么方法，只要当前栈内存中存在该变量，使用let/const重再复声明该变量就是语法错误
   var a = 1;
   let a = 2;   //报错 Uncaught SyntaxError: Identifier 'a' has already been declared
   console.log(a);
   ```

3. **let能解决typeof检测时出现的暂时性死区问题（let比var更严谨）；**

   暂时性死区：检测一个未被声明过的变量不会报错，结果是undefined；

   ```javascript
   console.log(typeof a);   //undefined
   //浏览器bug，本应该报错因为没有a(暂时性死区)
   ```

   ```javascript
   console.log(typeof a); //报错 Uncaught ReferenceError: Cannot access 'a' before initializatio
   let a;
   ```

4. **let创建的全局变量没有给window设置对应的属性；**

   ```javascript
   //全局作用域不带var：相当于给全局对象window设置了一个属性
   a = 10;    // 相当于 window.a = 10;
   console.log(a);  //10
   console.log(a==window.a)  //true
   console.log("a" in window)  //true
   
   //带var：在全局作用域下声明一个变量(全局变量),同时相当于给window增加了一个对应的属性(只有全局作用域具备这个特点)
   var b = 5;
   console.log(b);   //5
   console.log(window.b);  //5
   
   let c = 3;
   console.log(c);  //3
   console.log(window.c); //undefined
   
   //引申：
   //私有变量：形参赋值+变量提升+let/const
   
   //GO全局对象和VO(G)全局变量对象的关系
   //两者之间存在映射关系（创建一个全局变量也相当于给GO设置一个属性），除了基于let / const创建的变量;
   
   a=10;
   console.log(a);
   console.log(window.a);
   //此时a是GO的属性，相当于省略window，不能理解为全局变量
   
   //在全局上下文代码执行的时候，遇到一个变量，首先看是否为全局变量（如果是操作全局变量，var/function声明的也会给window设置一份），不是全局变量则继续看是否为GO的属性(如果是相当于省略window)，如果也不是则按照没有声明这个变量的错误处理；
   //在私有上下文中，如果不是私有的，则向上级上下文中查找，一直找到EC(G)为止，如果EC(G)中也没有：
   //1.如果是获取变量的值，则直接报错；
   //2.如果是设置变量的值，则相当于给window（GO）设置一个属性；
   
   //window.a是对象的成员访问，哪怕没有属性a,属性值为undefined，不会报错
   ```

5. **let会产生块级作用域；**


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
    /* 形参赋值 & 变量提升 & 词法解析
     * 报错原因：词法解析时已经知道在当前私有栈中有let a,此时出现的a都是私有的
     * 在当前作用域下(全局、私有、块作用域)，如果创建变量使用的是let/const等，一定不能在创建代码的前面使用这些变量，否则会报错Uncaught ReferenceError: Cannot access 'a' before initialization
     */
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

```javascript
//let所在大括号是一个块作用域（私有作用域）
if(1===1){
	var a = 12;   //没有块作用域，全局
    let b = 13;   //有块作用域，私有
}
console.log(a)   //12
console.log(b)   //报错 Uncaught ReferenceError: b is not defined
```

区别这么多，我想去静静。来点经典练习题看看是否真正掌握let和var的特点以及它们的区别吧！

```javascript
//练习1
var foo=1;
function bar(){
    //不管条件是否成立都要进行变量提升
    if(!foo){//!undefined->true
        var foo=10;
    }
    console.log(foo);   //10
}
bar();

//练习2
let foo=1;
function bar(){
    if(!foo){//!1->false
        let foo=10;
    }
    console.log(foo);   //1
}
bar();

//练习3
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
//练习4
let n=12;
~function(){
    if(1){
        let n=13;
    }
    console.log(n);  //12
}();

//练习5
let n=12;
~function(){
    if(1){
        n=13;
    }
    console.log(n);  //13
}();
console.log(n); //13

//练习6
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

```javascript
//练习7
/* 不管条件是否成立，都要进行变量提升
 * 第一步：var a;  //全局声明a相当于window.a  
 */
if(!("a" in window)){ //("a" in window) -> true -> !true = false -> 条件不成立
    var a = 1;
}
console.log(a);  //undefined
```

> **arguments**：函数内置的实参集合，箭头函数中没有arguments，不管是否定义了形参，也不管传递了多少实参，arguments中包含所有传递的实参信息（类数组集合），不传递实参是一个空的类数组集合；
>
> - 在js非严格模式下，arguments和形参存在映射关系（一个改都会跟着改）；
> - 严格模式下，arguments和形参的映射机制就切断了
>
> 映射机制是在函数代码执行前完成的，那会建立了映射就有映射，如果此时没建立，映射机制后续就不会再有了。

```javascript
/*	全局作用域
 *	1.变量提升  
 *      var a; function b(){}————关联堆内存AF0
 * 	2.代码执行
 *		var a = 4;   // a=> 4
 *		function b(){..}  //变量提升阶段已处理，不做处理
 *		a = b(1,2,3);   //执行b函数(传1,2,3),将函数返回值赋值给a——————————执行3
 */
var a = 4;
function b(x,y,a){
    /*	3.b函数执行形成私有作用域AF1
     *		3.1形参赋值 & 变量提升
     *			x:1  y:2  a:3
     *		3.2代码执行
     */
    //在js非严格模式下，arguments和形参存在映射关系（一个改都会跟着改）
    //console.log(arguments);   //{0:1,1:2,2:3,length:3...}
    console.log(a); //3
    arguments[2] = 10;//把传递的第三个实参修改为10，形参也跟着改为10
    console.log(a);//10
}
a = b(1,2,3);  //a = b的执行结果，无return，也就是a = undefined
console.log(a);//undefined


//严格模式下，arguments和形参的映射机制就切断了
"use strict";
function b(x,y,a){
    arguments[2] = 10;
    console.log(a);//3
}
b(1,2,3); 

//映射机制是在函数代码执行前完成的，那会建立了映射就有映射，如果此时没建立，映射机制后续就不会再有了。
var a = 4;
function b(x,y,a){
    a = 3;
    console.log(arguments[2]);   //undefined
}
a = b(1,2);  //传递2个参数，第三个无映射
```



