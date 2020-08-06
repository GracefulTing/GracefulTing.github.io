## Promise

Promise是ES6新增的内置类，是一个“承诺”设计模式，主要目的是用来解决JS异步编程中的回调地狱问题。

那什么是回调地狱呢？

现给出以下需求：等待1s输出1，再等待2s输出2，再等待3s输出3，然后结束。

```javascript
setTimeout(()=>{
    console.log(1);
    setTimeout(()=>{
        console.log(2);
        setTimeout(()=>{
            console.log(3);
        },3000)
    },2000)
},1000)
```

基本上就会这样去写代码。由上可见，在当前回调函数中嵌套新的回调函数，在新的回调函数中继续嵌套，这也就是所谓回调地狱。那么有什么解决方案呢？下面来尝试下reduce解决：

```javascript
let arr = [1000,2000,3000],
    i=0;
arr.reduce((n,item)=>{
    n = n + item;
    setTimeout(()=>{
        console.log(++i);
    },n)
    return n;
},0)
```



需求：设计一个等待函数，等待N时间后执行要做的事情。

```javascript
function delay(interval=1000){
    return new Promise(resolve=>{
        let timer = setTimeout(()=>{
            clearTimeout(timer);
            timer = null;
            resolve();
        },interval)
    })
}

delay(1000).then(()=>{
    console.log(1);
    return delay(2000);
}).then(()=>{
    console.log(2);
    return delay(3000);
}).then(()=>{
    console.log(3);
})

或者

(async function(){
    await delay(1000);
    console.log(1);
    await delay(2000);
    console.log(2);
    await delay(3000);
    console.log(3);
})()
```



> Promise是用来管控异步编程的，new Promise本身不是异步的，执行它的时候会立即把executor函数执行，只不过我们经常在executor中管控一个异步操作。
>
> executor函数中有两个默认的形参：resolve和reject函数；
>
> executor函数中一般用来管理一个异步编程（当然只写同步的也可以）；
>



每一个promise实例都包含的两个重要信息：

=>[[PromiseStatus]]：promise状态

- ​	pending —— 准备状态 
- ​	resolved(fulfilled) —— 成功状态
- ​	rejected —— 失败状态

=>[[PromiseValue]]：promise值（一般存放异步编程的处理结果）



resolve / reject 这两个函数的执行，目的是改变[[PromiseStatus]]和[[PromiseValue]]；一旦状态设置为成功或者失败，则不能在改变为其它的；resolve执行是成功，reject执行是失败；执行函数传递的结果就是[[PromiseValue]]；如果executor函数执行报错，状态也会变为失败状态，并且改变其值为失败原因。



```javascript
Promise.resolve(100);  //返回一个状态为成功，值为100的promise实例
Promise.reject(100);  //返回一个状态为失败，值为100的promise实例

//所有实例都成功，整体返回的promise实例才是成功，只要有一个失败，整体实例就是失败的
Promise.all([promise1,promise2,...]);    //=>返回一个新的实例
             
//多个promise实例同时进行，谁先处理完就以谁的状态作为最后的整体状态(不论是成功还是失败)
Promise.race([promise1,promise2,...]);   //=>返回一个新的实例
```

```javascript
function f1() {
    return Promise.resolve(1);
}

function f2() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(2);
      }, 2000)
    })
}

function f3() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(3);
      }, 1000)
    })
}

Promise.all([f1(), f2(), f3()]).then(values => {
    //values => array 按照顺序存储每一个实例返回的结果
    console.log(values);    //[1,2,3]
}).catch(reason => {
    //一旦有失败的，整体都是失败的，存储的是当前失败的实例的原因
    console.log(reason)
})

Promise.race([f2(), f3()]).then(values => {
    console.log(values);    //3
}).catch(reason => {
    console.log(reason)
})
```

resolve / reject 的执行是异步编程，需要等到then把方法存好之后，在根据状态通知then存放的某个方法执行；状态为成功后调用第一个回调函数执行，失败调用第二个回调函数执行；



-------

Promise中的异步指的是resolve/reject执行：

- 执行两个方法的时候不仅仅是修改状态和值，还要通知then存储的两个回调函数中的一个去执行；
- 执行两个方法的时候，需要先等待promise已经基于then把方法存储完毕，有方法后才回去执行；

------------



**每一次执行then会返回一个新的promise实例**；

```javascript
let p1 = new Promise((resolve,reject)=>{
    //下面两个方法同时存在，执行完resolve之后不会再执行reject
    resolve(100);    //=> 执行p2的方法A
    reject(100);     //=> 执行p2的方法B
})

let p2 = p1.then(result=>{
    //...方法A
},reason=>{
    //...方法B
})

let p3 = p2.then(result=>{},reason=>{})
```

p1的成功还是失败取决于executor函数中执行的是resolve还是reject或者执行是否报错。

不管p1.then中哪个方法执行，只要执行不报错，则p2的状态就是成功，只要报错，p2的状态就是失败；方法的返回值就是新实例的value值；

**特殊情况：如果p1.then中某个方法的执行返回的是一个新的promise实例，则新实例的状态和结果直接影响了p2的状态和结果；**



then注入回调方法的时候可以写也可以不写。

catch：

```javascript
p3.catch(reason=>{});
<==>
p3.then(null,reason=>{})   

null => 相当于
value=>{
    return Promise.resolve(value);
}
```

如果状态一旦确定要去执行then中的某个方法，但是此方法没有被注册，则顺延至下一个then的指定方法中【找下一个then中注册的对应方法】；所以一般我们用then来处理成功，catch来处理失败；

```javascript
Promise.resolve('ok').then(result=>{
    //...
}).catch(err=>{
    //...
})
```

finally：无论成功还是失败都会执行

```javascript
Promise.reject(100).then(value=>{
    ..
}).catch(err=>{
    console.log(err);
}).finally(_=>{
    //不管成功还是失败都会执行
})
```



## async和await

async和await是ES7中提供的，是对promise的一个补充（Promise语法糖）。

使用async修饰一个函数，函数返回的结果都会变为一个promise实例。

状态：大多都是成功的，如果代码执行报错返回失败；如果手动返回一个新的promise实例，则按照新实例的状态处理；

```javascript
async function fn(){
	return ...
}
```

使用await可以把一个异步任务变为类似于同步的效果（本质不是同步，还是异步，而且是异步中的微任务）。

**方法中想用await，必须把方法基于async修饰**。

```javascript
function f2() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(2);
      }, 2000)
    })
 }

 function f3() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(3);
      }, 1000)
    })
 }

(async function(){
    //1.先把fn2执行，观察fn2返回的是成功还是失败的promise
    //2.异步性：把当前上下文，await下面的代码整体都当作一个异步微任务，放置在EventQueue中
    //3.await只是处理promise实例是成功状态，如果返回状态是成功，则value是[[PromiseValue]]，并且把之前存储的异步任务拿到栈中让主线程执行
    //await处理后，立即把函数执行，哪怕函数立即返回成功或者失败的状态，await也没有把其立即处理，而是先等同步的都执行完才执行这些异步任务
    
    try{
        //如果fn2状态为失败的promise，则await下面的代码不再执行，浏览器抛出异常，所以需要基于try catch捕获异常，从而进行异常处理
    	let value = await fn2();  
    	console.log(value);
    }catch(e){
        
    }
    
})()
console.log(1);   //1  2
```



练习：

```javascript
// promise成功态，走向下一个then的成功态
read('./name.txt').then((data) => {
  return read(data)
}, (err) => {

}).then((data) => {
  console.log('------', data)  // 10
}, err => {
  console.log('---------', err+'错误')
})

// promise失败态，走向下一个then的失败态
read('./name.txt').then((data) => {
  return read(data+'1')
}, (err) => {

}).then((data) => {
  console.log('------', data)  // 10
}, err => {
  console.log('---------', err+'错误') // Error: ENOENT: no such file or directory, open 'age.txt1'错误
})

// 读取错误文件，执行reject(),当执行了错误状态的回调函数时，会默认返回一个undefined，普通值走向下一个then的成功方法
read('./name123.txt').then((data) => {
  return read(data)
}, (err) => {
   // return undefined
}).then((data) => {
  console.log('------', data)  // ------ undefined
}, err => {
  console.log('---------', err+'错误')
})
```



