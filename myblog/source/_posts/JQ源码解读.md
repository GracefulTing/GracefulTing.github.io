### 闭包源码解读

```javascript
typeof window != 'undefined' ? window : this;

/*
区分浏览器环境和NODE环境:
浏览器端运行传参：window
NODE环境传参：global或者一个模块
*/
```

思想：

1. 在闭包中把一些私有的信息暴露到全局使用：return  / window.xxx = xxx；
2. 在导入JQ但是并没有把自己的jquery/$暴露出去时，首先会把现有全局中叫做$/jquery的存储起来，防止自己后期设置的$/jquery会替换全局。
3. 基于noConflict转移JQ对$/jQuery的使用权。

commonJS规范



### JQ中的经典方法摘录

检测数据类型：

```javascript
function toType(obj){
    if ( obj == null ) {
		return obj + "";
	}
	return typeof obj === "object" || typeof obj === "function" ?
		class2type[ toString.call( obj ) ] || "object" :
		typeof obj;
}
```

检测是否为数组或者类数组：

```javascript
function isArrayLike( obj ) {
	var length = !!obj && "length" in obj && obj.length,
		type = toType( obj );
	if ( isFunction( obj ) || isWindow( obj ) ) {
		return false;
	}
	return type === "array" || length === 0 ||
		typeof length === "number" && length > 0 && ( length - 1 ) in obj;
}
```

检测是否为空对象：

```javascript
function isEmptyObject(obj){
    //保证obj是个对象
    if(!obj || toType(obj) !== 'object'){
        return false;   //非对象处理
    }
	for(var key in obj){
        if(!hasOwn.call(obj,key))   break;   //非私有属性
	    return false;
	}
	return true;
}
```

检测是否为纯粹的对象：直属原型链是Object.prototype

**数组不是纯粹对象**

```javascript
function isPlainObject(obj){
	var proto, Ctor;
	if ( !obj || toString.call( obj ) !== "[object Object]" ) {
		return false;
	}

	proto = getProto( obj );

	if ( !proto ) {
		return true;
	}
	Ctor = hasOwn.call( proto, "constructor" ) && proto.constructor;
	return typeof Ctor === "function" && fnToString.call( Ctor ) === ObjectFunctionString;
}
```

检测是否为函数：

```javascript
function isFunction(obj){
    return typeof obj === 'function' && typeof obj.nodeType !== 'number';
}
```

检测是否为window：

```javascript
function isWindow(obj){
    return obj != null && obj === obj.window;
}
```

