JQ源码解读

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