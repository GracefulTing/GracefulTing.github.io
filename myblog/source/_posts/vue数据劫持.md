### 数据劫持

> 定义：在访问或者修改对象的某个属性时，通过一段代码拦截这个行为，进行额外的操作或者修改返回结果。

提到数据劫持，就可以想到应用最广的双向绑定了，这也成为了面试必考题。之前对这块知识了解不够深入，趁着今天认真学了后，整理个笔记作为分享。Vue2.x中使用Object.defineProperty，Vue3.x后改用Proxy实现。

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

双花括号在初始渲染的时候，有可能会让用户看到小胡子



### v-cloak

专门用来解决双花括号问题，需要配合css使用；当vue编译完成之前，元素上会带有v-cloak行内属性，css样式就会起作用；当vue编译完成之后，会把这个指令删除，没有这个行内属性，那么对应的css也就不起作用。

```css
[v-cloak]{
    display: none;
}
```

### v-once

有这个指令的标签，vue只会编译一次。

### v-pre

优化性的指令，有这个指令的标签，vue不会编译这部分。

### v-for

理想的key应该为数据唯一的id，没有时可以使用索引，缺点：有时会导致元素不更新，复用了旧元素；



虚拟DOM：用JS对象模拟真实DOM





