## this

> 函数执行的主体，也就是谁执行函数，那么执行主体就是谁。

1. 给元素的某个事件绑定方法，当事件触发方法执行的时候，方法中的this是当前操作的元素本身。

   ```javascript
   var name = '张三';
   function fn(){
       console.log(this.name)
   }
   var obj = {
       name:'李四',
       fn:fn
   }
   obj.fn(); //李四   this -> obj
   fn();//张三   this -> window(非严格模式，严格模式是undefined)
   ```

2. 如何确定执行主体（this）是谁？当方法执行的时候，看方法前面是否有点，没有点this是window或者undefined；有点，点前面是谁this就是谁。

   ```js
   //情况1
   (function(){
       //自执行函数中的this是window或undefined
   })()
   
   //情况2
   let obj = {
       fn:(function(n){
           //把自执行函数执行的返回结果赋值给fn
           //this:window   
           return function(){
               //fn等于这个返回的小函数
               //this:obj
           }
       })(10)
   }
   obj.fn();
   
   //情况3
   function fn(){
       console.log(this);  //window
   }
   document.body.onclick=function(){
       fn(); //this:document.body
   }
   
   //情况4
   //hasOwnProperty方法中的this:arr.__proto__.__proto__
   arr.__proto__.__proto__.hasOwnProperty();
   ```

3. 

