# this call apply

## this
- this总是指向一个对象
- 具体指向的对象：是在运行时基于函数的执行环境动态绑定的，而非函数声明时的环境

### this的指向
1. 作为对象的方法调用
   - this指向该对象  
2. 作为普通函数调用
   - this指向全局对象，在浏览器里，是window对象
   - ECMAScript5的strict模式下，不会指向全局对象，而是undefined
3. 构造器调用
   - 构造器的定义和普通函数一样，区别在于被调用的方式
   - new运算符调用函数时，该函数总会返回一个对象：
      - 构造器没有显示返回任何数据，或者返回一个非对象类型的数据，构造器里的this指向返回的对象
      - 构造器显式地返回了一个object类型的对象，那么会返回这个对象， 而不是this
4. Function.prototype.call或Function.prototype.apply调用


### 丢失的this
正确：
```
var getId = function(id) {
  return document.getElementById(id);
}
getId('div1);
```
错误：
```
var getId = document.getElementById;
getId('div1');
```
> 分析：
> - 浏览器的js引擎的`document.getElementById`方法的内部实现中需要用到*this*。这个*this*本来被期望指向document，当getElementById方法作为document对象的属性被调用时，方法内部的this确实是指向document的。
> - 但当用getId来引用document.ElementById之后，再调用getId，此时就成了普通函数调用，函数内部的this指向了window，而不是原来的document。

更正：
```
var getId = function(id) {
  return document.getElementById.apply(document, id);
}
getId('div1');
```

### call和apply
- apply接收2个参数，第一个参数指定了函数体内this对象的指向，第2个参数为一个带下标的集合，这个集合可以是数组，也可以是类数组，apply方法把这个集合中的元素作为参数传递给被调用的函数
- call，可以清楚地表达形参和实参的对应关系
- 当传入的第一个参数为null，函数体内的this会指向默认的宿主对象，在浏览器中为window，ES6的strict模式下是null
- 有时候使用call或者apply的目的不在于指定this指向，而是另有用途，比如借用其他对象的方法，那么第一个参数传入null来代替某个具体的对象：`Math.max.apply(null, [1, 2, 3, 4, 5])`

### call和apply的用途
1. 改变this指向
2. Function.prototype.bind
> 简化版本：
```
Function.prototype.bind = function(context) {
  var self = this;
  return function() {
    return self.apply(context, arguments);
  }
}

var obj = {
  name: 'andy'
}

var func = function() {
  alert(this.name) // andy
}.bind(obj)

func()
```
> 预填参数：
```
Function.prototype.bind = function() {
  var self = this,
      context = [].shift.call(arguments), // 需要绑定this的上下文
      args = [].slice.call(arguments);  // 剩余的参数转成数组
  return function() {
    return self.apply(context, [].concat.call(args, [].slice.call(arguments)));
    // 组合两次分别传入的参数，作为新函数的参数
  }
}

var obj = {
  name: 'andy'
}

var func = function(a, b, c, d) {
  alert(this.name)  // andy
  alert([a, b, c, d]) // [1, 2, 3, 4]
}.bind(obj, 1, 2)

func(3, 4)
```
3. 借用其他对象的方法
> 函数的参数列表arguments是一个类数组对象，虽然有*下标*，但并非真正的数组，所以不能像数组一样，进行排序操作等，需要借用Array.prototype对象上的方法。
```
(function() {
  Array.prototype.push.call(arguments, 3)
  console.log(arguments) // [1, 2, 3]
})(1, 2)
```
> Array.prototype.push是一个属性复制的过程，可以把**任意**对象传入Array.prototype.push，**任意**对象需要目2个条件：
 - 对象本身要可以存取属性   // number类型是无法存取数据的
 - 对象的length属性可读写 // function的length是只读的
```
var obj = {};
Array.prototype.push.call(obj, 'first)
alert(obj.length)  // 1
alert(obj[0]) // first
```