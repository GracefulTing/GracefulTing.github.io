## this

> 函数执行的主体，也就是谁执行函数，那么执行主体就是谁。this是谁和函数在哪创建的或者在哪执行都没有必然的联系。

1. 给元素的某个事件绑定方法，当事件触发方法执行的时候，方法中的this是当前操作的元素本身。（特殊：IE6~8中基于DOM2事件绑定attachEvent方法中的this不是元素）

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

2. 如何确定执行主体（this）是谁？当方法执行的时候，看方法前面是否有点，没有点this是window(非严格模式)或者undefined(严格模式)；有点，点前面是谁this就是谁。

   自执行函数中的this是window或undefined。

   回调函数中this一般也是window / undefined（除非某个函数内部给回调函数做了特殊的处理）

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
       })(10)   //自治性函数只有给fn赋值时执行一次，后期执行obj.fn执行的是返回的小函数
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

3. 在构造函数模式执行中，函数体中的this是当前类的实例。

   ```javascript
   function Fn(){
       //this是f这个实例
   	this.name = 'xxx';
   }
   let f = new Fn();
   ```

4. 箭头函数中（私有块级上下文）没有自己的this，所用到的this都是其上级上下文中的this（也就是没有初始化this这一步）。

5. 基于call / apply / bind 可以强制改变this。

   

   练习-this指向（根据规则判断）

   ```javascript
   //练习1
   var fullName = 'language';
   var obj = {
       fullName:'js',
       prop:{
           getFullName:function(){
               return this.fullName;
           }
       }
   }
   console.log(obj.prop.getFullName())//undefined this->obj.prop->return obj.prop.fullName->undefined
   var test = obj.prop.getFullName;
   console.log(test());//"language" this->window->return window.fullName->"language"
   
   //练习2
   var name = 'window';
   var Tom = {
      name:'Tom',
      show:function(){
          console.log(this.name)
      },
      wait:function(){
          //this->Tom
          var fun = this.show;   //Tom.show
          fun(); //this->window->window.name->"window"
      }
   }
   Tom.wait();
   
   //练习3
   window.val = 1;
   var json = {
       val:10,
       dbl:function(){
           this.val*=2;
       }
   }
   json.dbl();  //this->json->json.val=20
   var dbl = json.dbl;
   dbl();//this->window->window.val=2;
   json.dbl.call(window);//通过call把this改为window->window.val=4;
   alert(window.val+json.val);//"24"
   
   //练习4
   (function(){
      //this ->  window
      var val = 1;
      var json = {
          val:10,
          dbl:function(){
              val*=2; //是一个变量，非私有，找上级作用域(栈)——全局作用域
          }
      }
      json.dbl();//this->json   => val=2
      alert(json.val+val);//"12"
   })()
   
   //练习5
   var foo={
       bar:function(){
           console.log(this);
       }
   }
   foo.bar();//this->foo
   (foo.bar)();//this->window
   ```

   

   综合练习

   ```javascript
   var num = 10;
   var obj = {
       num: 20
   };
   obj.fn = (function (num) {
       this.num = num * 3;
       num++;
       return function (n) {
           this.num += n;
           num++;
           console.log(num);
       }
   })(obj.num);
   var fn = obj.fn;
   fn(5);
   obj.fn(10);
   console.log(num, obj.num);
   ```

   解析：

   ```javascript
   /*
   * 1、全局作用域
   	1.1变量提升  var num,obj,fn;
   	1.2代码执行
   		num=10;------2.2执行后改为60
   		obj=AF0(obj对应的堆);
   		obj.fn=自执行函数执行的返回结果(把obj.num值作为实参传递给形参num)
   		自执行函数执行------执行2
   		2.2执行完之后obj.fn=BF0
   		fn=obj.fn;   //fn对应BFO
   		fn(5);       //fn(5)执行-----执行3
   		obj.fn(10);  //执行BF0,传参10------执行4
   		console.log(num,obj.num);//输出：65,30
   */
   var num = 10;
   var obj = {
       num:20
   }
   obj.fn=function(num){
       /*
       * 2、自执行函数执行-私有作用域 AAA
       	2.1形参赋值&变量提升  num=20;  -> 3.2执行后num=22;
       	2.2代码执行
       		this.num=num*3; //=> this->window => window.name=60(自执行函数this为window)->步骤1中num=60
       		num++;  //num=21(num私有)
       		return BF0; //(小函数堆)-形参+代码字符串
       */
       this.num = num*3;
       num++;
       return funtion(n){
           /*
           * 3、fn(5)执行即BF0(5)执行-私有作用域 BBB
           	3.1形参赋值&变量提升  n=5;
               3.2代码执行
               	this.num+=5;//this->window->window.name=65->修改1中全局num
               	num++;//num找上级作用域->AAA->AAA中num=21->修改2.1
               	console.log(num);//输出22
           */
           /*
           * 4、obj.fn(10)执行即BF0(10)执行-私有作用域 CCC
           	4.1形参赋值&变量提升 n=10;
           	4.2代码执行
           		this.num+=10;//this->obj->obj.num=30;修改全局obj.name
           		num++;//num是上级作用域AAA  num=23->修改2.1
           		console.log(num);//23
           */
           this.num+=n;
           num++;
           console.log(num)
       }
   }(obj.num);
   var fn = obj.fn;
   fn(5);   //22
   obj.fn(10);   //23
   console.log(num,obj.num);   //65  30
   ```


