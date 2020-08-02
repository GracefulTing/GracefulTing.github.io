## 进程和线程

进程：一个程序（浏览器新建一个页卡就是一个进程）  【工厂】

线程：一个进程中可能会包含多个线程，每个线程同时可以做一件事情  【工人】



## 同步异步编程

真正的同时做多件事情必须依赖多线程，浏览器是多线程的，其中包含：

1. GUI渲染线程；
2. HTTP网络请求线程（并发数 6-7）；
3. 事件监听、定时器监听...



任务队列是在打开页面的时候就分配了！！！



JS代码的运行是单线程的：浏览器只分配一个GUI渲染线程去执行JS代码。

**同步编程**——对于大部分js来讲，上面代码没有执行完下面代码是不能执行的；

**异步编程**——上一件事情没有完成，把它做一些特殊处理，下一件事情继续执行；某些JS代码（事件绑定、定时器、Promise / async / await / ajax等），需要在上面代码没有处理完的情况下，GUI渲染线程能够继续向下执行；【但是绝对不是JS可以同时处理两个事情】

- setTimeout / setInterval定时器；
- 事件绑定/事件监听；
- ajax/fetch请求数据的时候也是采用异步的（ajax可以控制同步）;
- promise中的异步操作；
- async/await中的异步操作；
- node.js中的异步操作；

```javascript
/*	度量一个程序的执行时间
 *	事后统计法——time/timeEnd可以记录一段程序执行的时间，受电脑性能和执行时候的换接状态影响;
 *  事前分析估算方法——时间复杂度O
 */
console.time('AAA');
...
console.timeEnd('AAA');
```

```javascript
//定时器设置的0ms也不是立即执行的同步任务，浏览器有最小等待时间，它还是异步任务，需要放在EventQueue中
setTimeout(()=>{
    ...
},0) 

//练习
console.log(1);
setTimeout(()=>{
    console.log(2);
},1000)
console.log(3);
setTimeout(()=>{
   console.log(4); 
},0)
console.log(5);
for(let i=0;i<99999999;i++){
    if(i===99999998){
        console.log(6);
    }
}
console.log(7);
//输出结果：1 3 5 6 7 4 2

//解析
GUI渲染线程：自上而下执行代码
=> 1
=> 设置定时器(1s后执行函数A)——异步->放到事件队列/任务队列（Event Queue）——任务1：1s后执行函数A,浏览器会开辟一个监听时间的线程来监听时间是否到达;
=> 3
=> 设置定时器(5~6ms执行函数B)——异步->放到事件队列/任务队列（Event Queue）——任务2：5~6ms后执行函数B,监听
=> 5
=> 循环执行(300~400ms)——同步
	循环输出6，循环过程中，任务队列中的任务2已经到达执行时间，但是此时GUI渲染线程还没有自上而下渲染完，所以需要所有任务队列中的任务（哪怕到达时间的）继续等待;
=> 7
-------------------------------------空闲---------------------------------------------
此时，GUI渲染线程自上而下执行完，空闲下来了。到任务队列中查找到达时间的能执行的放到GUI中去执行。如果有多个到达时间，谁先到谁先执行。
任务2执行(GUI忙)
=> 4
    代码执行完，GUI空闲，继续去任务队列中查找
....
Event Loop事件循环机制
```

总结：

1. GUI渲染中遇到异步操作，都会先放在事件队列中；

2. JS是单线程的，一次执行做一件事情，所以GUI只要没有闲下来，就不能干其它事情（哪怕任务队列中的某些任务到达执行时间，也不会立即执行，需要等GUI闲下来）；

   定时器设置的等待时间是 最快的执行时间，到达时间后不一定能执行，需要看GUI有没有闲下来。比如饭店只有一个服务员，忙着给别的客人点菜，即使你的饭出来也不能端，只有闲下来才可以。

3. 任务队列中的任务，最后也是拿到GUI线程中处理的，一个任务处理完才能继续处理下一个任务；



练习：

```javascript
for(var i=0;i<5;i++){
	setTimeout(()=>{
		console.log(i);   
	},0)
}
//输出结果：5 5 5 5 5

//第一次循环：向任务队列插入一个定时器
//第二次循环：...
..
//第五次循环：...
//循环结束【GUI空闲】  全局i=5 任务队列中有5个定时器
//定时器执行中遇到i，不是自己的则找全局的，输出5次5
```

```javascript
setTimeout(()=>{
    console.log(1);
    while(1===1){}
},10)
console.log(2);
for(let i=0;i<90000000;i++){
   ... //79ms
}
//循环结束时，任务1到达时间，任务1放到GUI中渲染，遇到死循环（GUI永远忙不完），所以4不执行
console.log(3);
setTimeout(()=>{
   console.log(4); 
},5)
console.log(5);
//输出结果：2 3 5 1
```



## 微任务和宏任务

1、微任务优先级永远高于宏任务

2、宏任务的优先级按照谁先达到执行的时间

微任务：Promise / async / await

宏任务：事件绑定 / 定时器 / ajax异步请求 / GUI渲染任务局

先找微任务，只要还有微任务，则继续查找，一直到没有微任务了，才去宏任务队列中查找。

```javascript
await async2();
console.log(123);
//await是异步微任务，async2()先执行，当async返回成功状态后，把下面代码执行

new Promise(function(resolve){
	..
	resolve();
}).then(function(){
	..
})
//promise方法立即执行，resolve()是异步微任务，等待then把成功失败的方法放到池子里再通知方法执行
//resolve()控制then存放的方法执行
```



练习1：

```javascript
async function async1() {
	console.log('async1 start');
	await async2();
	console.log('async1 end');
}
async function async2() {
	console.log('async2');
}
console.log('script start');
setTimeout(function () {
	console.log('setTimeout');
}, 0)
async1();
new Promise(function (resolve) {
	console.log('promise1');
	resolve();
}).then(function () {
	console.log('promise2');
});
console.log('script end');
```

一张图带你看整个流程：

![](/img/blog/render/宏任务和微任务.png)



练习2：

```javascript
async function async1() {
    console.log('async1 start1');
    async2();
    console.log('async1 end');
}

async function async2() {
    console.log('async2');
}

console.log('illegalscript start');

setTimeout(function () {
    console.log("setTimeout")
});

async1();

new Promise(function (resolve) {
    console.log('promise1');
    resolve()
}).then(function () {
    console.log('promise2');
});

console.log('illegalscript end')
```

答案：

```
illegalscript start
async1 start
async2
async1 end
promise1
illegalscript end
promise2
setTimeout
```

解析过程：

![](/img/blog/render/时间循环机制.png)