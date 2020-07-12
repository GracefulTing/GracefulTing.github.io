## 数据类型

**基本数据类型**：number    string    boolean   null    undefined    symbol     bigint

```javascript
//symbol —— 不能new ，每次创建都是一个唯一值
Symbol(10)===Symbol(10)   //false

bigint：
Number.MAX_SAFE_INTEGER 最大安全数
Number.MIN_SAFE_INTEGER 最小安全数
超过最大安全数后面＋n  类型为bigint

//Infinity 正无穷大
//NaN:not a number   不是一个有效数字，但是属于number类型
//NaN===NaN 为false，NaN和自己或者任意值都不相等
//isNaN([val]):把[val]隐式转化(通过Number[val]转换)为数字类型，在看一下是否为非有效数字，如果确实是非有效数字，结果返回true，反之如果是有效数字，返回结果为false
//NaN+数字=NaN   NaN+字符串=字符串
```

**引用数据类型**：object   function

```javascript
object : new Array   /   new  RegExp  /  new  Date ...
```

${}存放的是js表达式：执行有返回结果的就是js表达式（变量 / 数学运算 / 三元运算符...）

```vue
let str = `
	<ul>
		${data.map(item=>{
            return `<li>${item.title}</li>`
        }).join('')}
	</ul>
`;
```

## 检测数据类型

4种方法：typeof  /  instanceof / constructor / Object.prototype.toString.call([val])

{} / [] / null / 正则  —— typeof  都是object

```javascript
typeof
  a、检测返回的结果是一个字符串，字符串种包含对应的数据类型；
  b、typeof null=>'object'   null不属于对象，而是因为二进制存储值以000开头了；检测对象细分的类型也都是'object'
  c、检测数组、正则、日期、对象返回的都是object，所以无法基于此方法细分对象；
  
let a = typeof typeof typeof [12,23];
console.log(a)   //'string'
```

基于alert输出一个值，都要把值隐式转换为字符串然后输出。alert输出的结果都会转换为字符串



## 数据类型转换

**把其他数据类型转换为Number类型**：Number / parseInt / parseFloat

```javascript
/*********************Number规则**********************/
Number(val)==0,其中val可以为：''  false  null  []

//把字符串转换为数字，要求字符串中所有字符都必须是有效数字才能转换
console.log(Number(''));  //0
console.log(Number('5')); //10
console.log(Number('5px')); //NaN----px属于非有效数字

//布尔值转换为数字
console.log(Number(true));  //1
console.log(Number(false));  //0

//把空转换为数字
console.log(Number(null));  //0
console.log(Number(undefined));  //NaN

//把Symbol/BigInt转换为数字
console.log(Number(Symbol(10)));  //报错----不允许转换
console.log(Number(BigInt(10)));  //10

//把对象或者函数转换为数字，要先用toString把对象转换为字符串，再把字符串转换为数字
console.log(Number({
    0:10
}));//NaN   普通对象toString()是检测数据类型，结果[object Object]=>转换数字NaN
console.log(Number([20]));//20  数组.toString()=>"20"
console.log(Number([1,2]));//NaN  数组.toString()=>"1,2"
console.log(Number([]));//0  数组.toString()=>""


/******************parseInt/parseFloat规则**************************/
//parseInt/parseFloat([val])：先把val转换为字符串，再从字符串左边第一个字符开始查找，把所有找到的有效数字变为数字，直到遇到一个非有效数字则停止查找，如果一个有效数字字符都没找到，返回结果NaN；
//parseFloat只是比parseInt多能识别一个小数点而已。
console.log(parseInt("10px20")); //10
console.log(parseInt('10.3px'));//10
console.log(parseFloat('10.3px'));//10.3
console.log(parseInt("width:10px"));//NaN
```

**把其他类型转换为字符串**：toString() / String

```javascript
/**********************toString()********************/
//其他类型转换为字符串，一般都是直接""包起来，只有{}普通对象调用toString是检测数据类型，因为调用了Object.prototype.toString，返回结果："[object Object]"，转换为数字NaN。

//在加号运算时，如果加号某一边出现字符串，则不是数学运算，而是字符串拼接;
//基于alert/confirm/prompt/document.write...输出内容时，都要先把内容转换为字符串再输出;

let result = 10 + false + undefined + [] + 'Tencent' + null + true + {};
//=>10+0=>10+NaN=>NaN+""=>"NaN"+'Tencent'=>"NaNTencent"+后面都是字符串=>"NaNTencentnulltrue[object Object]"
console.log(result);       //"NaNTencentnulltrue[object Object]"
```

***注意***：加号即使一边出现字符串或者对象，也不一定是字符串拼接，比如   ++i/i++/+i —— 数学运算，如下：

```javascript
let n = '10';
console.log(++n);  //11  => 转换类型+计算
console.log(typeof ++n)  //number

let n = '10';
console.log(+n);   //10  => 只转换类型
console.log(typeof +n)   //number

console.log({}+0);  //0  {}认为是代码块，不参与运算=>只处理+0=>0

console.log(({}+0));//'[object Object]0'        {}参与运算

console.log(0+{});//'0[object Object]' 
```

**把其它数据类型转换为布尔**

```javascript
/*
 *  1. ! 转换为布尔值后取反;
 *  2. !! 转换为布尔值;
 */
//规则：只有0、NaN、null、undefined、空字符串这5个值会转换为false，其余都是true;
console.log(!!({}));  //true
```

JS中的比较操作：==   ===

==：如果左右两边数据类型不一致，==会默认把数据类型转换为一致的再去比较；

===：如果左右两边数据类型不一致直接返回false，因为它要求数据类型和值都一样才相等；

```javascript
/*******************==比较规则***********************/
//类型一样的特殊点
{}=={}   //false   对象比较的是堆内存地址
[]==[]   //false
NaN==NaN   //false   NaN和谁都不相等

//类型不一样的转换规则
1.null==undefined   //true   剩下null/undefined和其他任何数据类型值都不相等
2.字符串==对象，要先把对象转换为字符串
3.如果==两边数据类型不一致，都需要先转换为数字再比较

console.log([] == false); // true  
// 规则：比较过程中，两边数据值不一样，都转换为数字再比较
// 左边[] =====> '' ====> 0
// 右边 false  => 0

console.log(![] == false); // true
// 规则：先算![] => false
// false == false
```



看完上面的规则，再来练个手呗！

练习：

```javascript
//练习1
parseInt("")   //NaN      非有效数字
Number("")     //0
isNaN("")      //false        ===> isNaN(Number('')) => isNaN(0) => false

//练习2
parseInt(null)    //NaN       ===> parseInt("null") => NaN
Number(null)   //0
isNaN(null)    //false        ===> isNaN(Number(null)) => isNaN(0) => false

//练习3
parseInt("12px")    //12
Number("12px")      //NaN
isNaN("12px")       //true
parseFloat("1.6px") + parseInt("1.2px") + typeof parseInt(null);  //"2.6number"
//1.6+1+"number" => "2.6number"

//练习4
isNaN(Number(!!Number(parseInt("0.8"))))   //false
//parseInt('0.8') => 0
//!!Number(0) =>  false
//Number(false) => 0
//isNaN(0) =>  false

//练习5
typeof !parseInt(null) + !isNaN(null);    ////"booleantrue"
//!parseInt(null) => !NaN => !false => true
//typeof !parseInt(null) => "boolean"
//!isNaN(null) => !false => true
```



parseInt练习题：

```javascript
let arr = [10.18,0,10,25,23];
arr = arr.map(parseInt);
console.log(arr);  //[10,NaN,2,2,11]

/******************解析***********************/
/*	arr.map((item,index)=>{
 *		//循环遍历数组中的某一项触发回调函数，item-当前项   index-当前项索引
 *  })
 *  此时parseInt,所以=>
 *  从字符串左侧第一个字符开始查找，找到符合[radix]进制的值，如果遇到一个不合法的，则停止查找，把找到的值变为数字，在按照把[radix]转换成十进制的规则处理.
 *  parseInt('10.18',0);  => 10
 *  parseInt('0',1); => NaN
 *  parseInt('10',2); => 10 => 1*2^1+0*2^0=2
 *  parseInt('25',3); => 2 => 2*3^0 = 2
 *  parseInt('23',4); => 23 => 2*4^1 + 3*4^0 = 11
 *  parseInt([value],[radix]):把value看做n进制,最后转换为10进制。第二个参数是一个进制，不写或者写0都默认按照10进制处理(特殊情况：如果以0X开头，  默认值是16进制).进制取值范围为2~36之间，如果不在这之间，整个程序运行结果一定是NaN;
 */

//把一个值转换为十进制  //位权值，个位0  十位1  ...
147(八进制) => 十进制        //1*8^2 + 4*8^1 + 7*8^0 
12.23(四进制) => 十进制      //1*4^1 + 2*4^0 + 2*4^-1 + 3*4^-2
```

