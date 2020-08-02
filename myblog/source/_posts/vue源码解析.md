rollup.config.js



options Api：通过一个选项进行配置



**响应式数据原理**

数据变化视图会更新，视图变化数据会被影响  

MVVM----不能跳过数据去更新视图

vue不是MVVM框架，$ref



**数据劫持**

数据劫持——重写方法，通知视图更新

默认会递归去调用define.property进行拦截，性能差 => proxy



vue.$set原理：splice



判断一个对象是否被观测过看它是否有__ ob__ 这个属性

```javascript
//防止对象被重复劫持    使用方法
Object.definedProterty(value,"__ob__",{
    enumberable:false,  //不能被枚举,不能被循环出来
    configurable:false, 
    value:this
})
```



**html解析成ast语法树**

ast解析template => render函数

ast可以用来描述代码的：js/css/dom

虚拟dom：用对象来描述节点；



```javascript
//html模板 => render函数  （ast是用来描述代码的）
//1.需要将html代码转化成ast语法树，可以用ast树来描述语言本身

let ast = parseHTML(template);

//2.优化静态节点

//3.通过这棵树 重新生成的代码
let code = generate(ast);

//4.将字符串变为函数
let render = new Function(`with(this) {return ${code}}`);  //限制取值范围，通过with来取值，稍后调用render函数就剋通过改变this，让这个函数内部取到结果了
----理解：with包裹，this作用域取值
```

?:   匹配不捕获



**html => ast 过程**：把标签转换为对象，使用正则去匹配，分别赋值对象的标签tag，属性attr，children，parent等，每次匹配到后就删除字符，继续匹配直到匹配完。

标签闭合，确立父子关系。开始时创建一个元素作为根元素，把当前解析的标签存起来，将生成的ast元素进栈。结尾标签处取出栈中最后一个，创建父子关系。

**ast => render过程**：转换，el.tag容易，el.attrs做处理，把属性转换为key:value形式。判断是否有儿子节点，有的话取出拼接在字符串；



mountComponent:

调用render方法渲染 el属性

```javascript
//先调用render方法创建虚拟节点，再将虚拟节点渲染到页面上
vm._update(vm._render());

Vue.prototype._update = function(vnode){
    const vm = this;
    patch(vm.$el,vnode);   
}

//将虚拟节点转换成真实节点
function path(oldVnode,vnode){
    let el = createElm(vnode);   //根据虚拟节点创建真实dom
    let parentElm = oldVnode.parentNode;  //获取老节点的父节点---body
    parentElm.insertBefore(el,oldVnode,nextSibling);  //当前真实插入到老节点后面
    parentElm.removeChild(oldVnode);  //删除老节点
}
```



vue的渲染流程：先初始化数据 => 将模板进行编译 => render函数 => 生成虚拟节点 => 生成真实dom => 呈现在页面上

