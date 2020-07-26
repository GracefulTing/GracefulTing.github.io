在JS语言中，它的继承和其它编程语言不太一样。

继承的目的：让子类的实例同时也具备父类中私有的属性和公共的方法。

下面是JS的常见继承方式：

### 原型继承：

> 原型继承 => 子类的原型等于父类的实例；
>
> - 子类的实例，能够用子类私有的和原型上公有的；
> - 父类的实例，能够用父类私有的和原型上公有的；

```javascript
//父类
function Parent(){
    this.x = 1;
}
Parent.prototype.getX = function getX(){
    return this.x;
}

//子类
function Child(){
    this.y = 2;
}
Child.ptototype = new Parent;   //原型继承
Child.prototype.getY = function getY(){
    return this.y;
}

let c = new Child();
console.log(c);
```

原型继承特点：

1. 父类中私有和公有的属性方法，最后都变为子类实例公有的；
2. 和其它语言不同的是，原型继承并不会把父类的属性方法 "拷贝" 给子类，而是让子类实例基于__ proto __原型链找到自己定义的属性和方法；------指向/查找的方式

```javascript
//子类重写父类方法
c.__proto__.xxx = xxx  
//修改子类原型（原有父类的一个实例）中的内容，内容被修改后，对子类的其它实例有影响，但是对父类的实例不会有影响


c.__proto__.__proto__.xxx = xxx
//直接修改的是父类原型，这样不仅会影响其它父类的实例，也影响其它子类的实例
```

```javascript
Child.prototype.__proto__ === Parent.prototype;
```

### CALL继承

只能继承父类中私有的，不能继承父类中公共的。

```javascript
//父类
function Parent(){
    this.x = 1;
}
Parent.prototype.getX = function getX(){
    return this.x;
}

//子类
function Child(){
    //在子类构造函数中，把父类当作普通函数执行（没有父类实例，父类原型上的那些东西和它也就没关系了）
    Parent.call(this);  //this执行Child实例c,this.x=1相当于强制给c这个实例设置一个私有属性x:1,相当于让子类的实例继承了父类的私有属性，并且也变为了子类私有的属性  ---拷贝式
    this.y = 2;
}
Child.prototype.getY = function getY(){
    return this.y;
}

let c = new Child();
console.log(c);
```

那么有方式可以让 “ 父类私有变为子类私有的，父类公有变为子类公有 ” 的吗？

### 寄生组合式继承

【CALL继承 +  另类原型继承】

```javascript
//父类
function Parent(){
    this.x = 1;
}
Parent.prototype.getX = function getX(){
    return this.x;
}

//子类
function Child(){
    Parent.call(this);  
    this.y = 2;
}
//Child.prototype.__proto__ === Parent.prototype;  -----IE不支持__proto__
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
Child.prototype.getY = function getY(){
    return this.y;
}

let c = new Child();
console.log(c);
```

### ES6中的类和继承：extends（类似于寄生组合继承）

> 继承后一定要在constructor第一行加上super

```javascript
Class Parent{
    constructor(){
        this.x = 100;
    },
    //Parent.prototype.getX = function ...
    getX(){
        return this.x;
    }
}

//继承：extends Parent
class Child extends Parent{
    constructor(){
        super();  //-----------> 类似于之前的call继承
        //super(100,200)——相当于把Parent中的constructor执行，传递了100和200
        this.y = 200;
    },
    getY(){
        return this.y;
    }
}

Child();   //报错-----ES6中创建的类，不能当作普通函数执行，只能new执行

let c = new Child;
console.log(c);
```



