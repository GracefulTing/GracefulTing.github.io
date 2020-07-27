### Object.defineProperty

该方法不兼容IE8

Object.defineProperty：挨个对属性进行数据劫持；

```javascript
var obj = {};

var temp = 123;
//数据劫持
Object.defineProperty(obj,'name',{
    get(){
        //当name被调用时触发，调用时获取到的值是这个函数的返回值
        console.log('get');
        return temp;
    },
    set(val){
        //当name属性被赋值时触发，val对应的是给name赋的值
        console.log('set',val);
        temp = val;
    }
})
```

### Proxy

```javascript
var obj = {
    name:'张三',
    age:18
}
var tempObj = new Proxy(obj,{
    get(target,key){
        //target--->obj
        //key--->调用的对应属性的名字
        console.log('get',arguments);
        return target[key];
    },
    set(target,key,val){
        console.log('set');
        target[key] = val;
    }
})
```

proxy：代理，tempObj相当于obj的秘书。

优点：不用对每个属性挨个进行劫持，批量来处理；

缺点：不兼容；



### v-text

优势：死活不会显示小胡子

{{ }} 在初始渲染的时候，有可能会让用户看到小胡子



### v-cloak

专门用来解决{{ }}问题，需要配合css使用；当vue编译完成之前，元素上会带有v-cloak行内属性，css样式就会起作用；当vue编译完成之后，会把这个指令删除，没有这个行内属性，那么对应的css也就不起作用。

### v-once

有这个指令的标签，vue只会编译一次。

### v-pre

优化性的指令，有这个指令的标签，vue不会编译这部分。







