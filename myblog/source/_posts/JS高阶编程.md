### 单例设计模式

> 概念：用单独的实例来管理当前事务的相关特征（属性和方法），类似于实现一个分组的特点，而此时obj1和obj2不仅仅叫做一个对象，也被称为命名空间。

基于闭包管控的单例模式称为：高级单例模式，以此来实现早期的模块划分（最早的模块化思想）；

```javascript
let module = (function(){
    function query(){}
    function tools(){}
    return {
        name:'test',
        tools:tools
    }
})();
```



### 惰性思想  （函数重构）

```javascript
//下面方法只判断一次，不需要执行几次就重复判断几次
function getCss(el,attr){
    //'name' in obj——检测当前属性name是否为obj的一个属性
    if('getComputedStyle' in window){
        getCss = function (el,attr){
            return window.getComputedStyle(el)[attr];
        }
    }else{
        getCss = function (el,attr){
            return el.currentStyle[attr];
        }
    }
    return getCss(el,attr);
}

getCss(document.body,'margin');
getCss(document.body,'padding');
getCss(document.body,'width');
```



### 柯理化函数

> 概念：利用闭包的保存机制，把一些信息预先存储下来（预处理的思想）；

```javascript
let res = fn(1,2)(3);
console.log(res); //=>6  1+2+3
```

解析：fn(1,2)先执行，紧接着执行..(3)，那么fn(1,2)需要返回一个函数去执行，所以return function(){...}

```javascript
function fn(...outer){
    return function anonymous(...inner){
        let params = outer.concat(inner);  //外层和里层函数传递的所有值都合并在一起
        return params.reduce((n,item)=>{
           return n + item; 
        },0)
    }
}
```

知识延申：reduce

```javascript
//不传递第二个实参值
let arr = [1,2,3,4,5];
let res = arr.reduce((n,item)=>{
    console.log(n,item);   
    //第一次输出：1  2    第一次触发回调函数执行，n是第一项，item是第二项
    //第二次输出：3  3    第二次触发回调函数执行，n是上一次回调函数的返回结果，item继续向后遍历数组项
    //第三次输出：6  4    ...
    //第四次输出：10 5    第 m 次触发回调函数执行，n是 m-1 次回调函数的返回结果
    return n+item;  //return信息会作为下一次回调函数执行n的结果
})


//传递第二个实参值
let arr = [1,2,3,4,5];
let res = arr.reduce((n,item)=>{
    console.log(n,item);   
    //第一次输出：0  1    第一次触发回调函数执行，n是第二个参数值，item是数组第一项
    //第一次输出：1  2    第二次触发回调函数执行，n是上一次回调函数的返回结果，item继续向后遍历数组项
    //第二次输出：3  3    ...
    //第三次输出：6  4    ...
    //第四次输出：10 5    第 m 次触发回调函数执行，n是 m-1 次回调函数的返回结果
    return n+item;  //return信息会作为下一次回调函数执行n的结果
},0)
```

回调函数：把一个函数作为值传递给另外一个函数，在另外一个函数中把这个函数执行（这是实现函数式编程重要的知识）；

- 函数式编程：注重结果，不在乎过程，过程交给别人处理；（利用reduce去累加）
- 命令式编程：注重过程，需要自己去实现过程；(自己去编写累加)



### compose思想

compose组合函数，把多层函数嵌套调用扁平化。

```javascript
/*
  ---------------------------------------------问题描述---------------------------------------------
  在函数式编程当中有一个很重要的概念就是函数组合， 实际上就是把处理数据的函数像管道一样连接起来， 然后让数据穿过管道得到最终的结果。 例如：
    const add1 = (x) => x + 1;
    const mul3 = (x) => x * 3;
    const div2 = (x) => x / 2;
    div2(mul3(add1(add1(0)))); //=>3

    而这样的写法可读性明显太差了，我们可以构建一个compose函数，它接受任意多个函数作为参数（这些函数都只接受一个参数），然后compose返回的也是一个函数，达到以下的效果：
    const operate = compose(div2, mul3, add1, add1)
    operate(0) //=>相当于div2(mul3(add1(add1(0)))) 
    operate(2) //=>相当于div2(mul3(add1(add1(2))))

    简而言之：compose可以把类似于f(g(h(x)))这种写法简化成compose(f, g, h)(x)，请你完成 compose函数的编写 
*/


/*
	------------------------------------------分析-------------------------------------------------
	compose函数执行后还可以继续执行，那么compose函数返回的是一个函数。
*/

const fn1 = x => x+10;
const fn2 = x => x-10;
const fn3 = x => x*10;
const fn4 = x => x/10;

function compose(...funcs){
    //funcs:存储按照顺序执行的函数(数组) ------ [fn1,fn3,fn2,fn4]
    return function anonymous(...args){
        //args:存储第一个函数执行需要传递的实参信息(数组) ------ [10]
        
        if(funcs.length===0) return args;
        if(funcs.length===1) return funcs[0](...args);
        return funcs.reduce((n,item)=>{
           //第一次n:第一个函数执行的实参   item是第一个函数
           //第二次n:上一次item执行的返回值，作为实参传递给下一个函数执行
           return Array.isArray(n) ? item(...n) : item(n) ;
        },args)
    }
}

let res = compose(fn1,fn3,fn2,fn4)(10);
console.log(res);
```

