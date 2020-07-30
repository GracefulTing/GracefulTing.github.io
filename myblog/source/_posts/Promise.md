### Promise

在new Promise的同时就会把executor函数执行；

executor函数中有两个默认的形参：resolve和reject函数；

executor函数中一般用来管理一个异步编程（当然致谢同步的也可以）；



每一个promise实例都包含的两个重要信息：

=>[[PromiseStatus]]：promise状态

- ​	pending —— 准备状态 
- ​	resolved(fulfilled) —— 成功状态
- ​	rejected —— 失败状态

=>[[PromiseValue]]：promise值（一般存放异步编程的处理结果）



resolve / reject 这两个函数的执行，目的是改变[[PromiseStatus]]和[[PromiseValue]]；一旦状态设置为成功或者失败，则不能在改变为其它的；resolve执行是成功，reject执行是失败；执行函数传递的结果就是[[PromiseValue]]；

```javascript
Promise.resolve(100);  //创建一个状态为成功，值为100的promise实例
Promise.reject(100);  //创建一个状态为失败，值为100的promise实例

//所有实例都成功，整体返回的promise实例才是成功，只要有一个失败，整体实例就是失败的
Promise.all([promise1,promise2,...]);  
             
//多个promise实例同时进行，谁先处理完就以谁的状态作为最后的整体状态(不论是成功还是失败)
Promise.race([promise1,promise2,...]);
```

resolve / reject 的执行是异步编程，需要等到then把方法存好之后，在根据状态通知then存放的某个方法执行；

每一次执行then会返回一个新的promise实例；



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

p1的成功还是失败取决于executor函数中执行的是哪个方法。

不管p1.then中哪个方法执行，只要执行不报错，则p2的状态就是成功，只要报错，p2的状态就是失败；

如果p1.then中某个方法的执行返回的是一个新的promise实例，则新实例的最后结果直接影响了p2的结果；

catch：

```javascript
p3.catch(reason=>{});
<==>
p3.then(null,reason=>{})   
```



如果then中某个方法没有写，则顺延至下一个then的指定方法中；所以一般我们用then来处理成功，catch来处理失败；

```javascript
Promise.resolve('ok').then(result=>{
    //...
}).catch(err=>{
    //...
})
```

