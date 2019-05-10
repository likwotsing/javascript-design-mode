# 03章 闭包和高级函数

## 闭包
- 变量的作用域：变量的有效范围
- 变量的生存周期：
```
var Type = {};
```
```
for(var i = 0, type; type = ['String', 'Array', 'Number', 'Boolean', 'Function', 'Object', 'Null', 'Undefined'][i++];) {
  (function(type) {
    Type['is' + type] = function(obj) {
      return Object.prototype.toString.call(obj) === '[object ' + type + ']';
    }
  })(type)
}
```
```
for(let i = 0, type; type = ['String', 'Array', 'Number', 'Boolean', 'Function', 'Object', 'Null', 'Undefined'][i++];) {
  Type['is' + type] = function(obj) {
    return Object.prototype.toString.call(obj) === '[object ' + type + ']';
  }
}
```
```
console.log(Type.isArray([]))
console.log(Type.isString('123'))
console.log(Type.isBoolean(false))
console.log(Type.isFunction(function(){}))
console.log(Type.isObject({}))
```

### 闭包的作用
1. 封装变量
> 把一些不需要暴露在全局的变量封装成*私有变量*
```
var mult = (function() {
  var cache = {};
  var calculate = function() {
    var a = 1;
    for(var i = 0, l = arguments.length; i < l; i++) {
      a = a * arguments[i];
    }
    return a;
  }
  return function() {
    var args = [].join(arguments, ',');
    if(args in cache) {
      return cache[args]
    }
    return cache[args] = calculate.apply(null, arguments);
    // return cache[args] = calculate(arguments);
    // 注意和使用apply的区别
  }
})()
```
2. 延续局部变量的寿命

### 闭包与内存管理
- 使用闭包的一部分原因是我们选择主动把一些变量封闭在闭包中，因为可能在以后还需要使用这些变量，把这些变量放在闭包中和放在全局作用域，对内存的影响是一致的，并不能说成是内存泄漏。
- 闭包和内存泄漏有关系的地方是：使用闭包的同时比较容易形成循环引用，如果闭包的作用域链中保存着一些DOM节点，这时候就有可能造成内存泄漏。但这本身并非闭包的问题，也并非JavaScript的问题。在IE浏览器中，由于BOM和DOM中的对象是使用C++以COM(Component Object Model，组件对象模型)对象的方式实现的，而COM对象的垃圾收集机制采用的是引用计数策略。在基于引用计数策略的垃圾回收机制中，如果两个对象之间形成了循环引用，那么这两个对象都无法被回收，但循环引用造成的内存泄漏在本质上也不是闭包造成的。
- 如果要回收闭包中封闭的变量，手动把这些变量设置为null。将变量设置为null意味着切断变量与它此前引用的值之间的连接。当垃圾收集器下次运行时，就会删除这些值并回收它们占用的内存。

## 高阶函数

> 高阶函数是指至少满足下列条件之一的函数：
  - 函数可以作为参数被传递
  - 函数可以作为返回值输出

### 函数作为参数传递
1. 回调函数
2. Array.prototype.sort

### 函数作为返回值输出
1. 判断数据的类型
2. getSingle：单例模式

## 高阶函数的其他应用
1. currying
  - 函数柯里化，又称*部分求值*
  - 一个currying的函数首先会接收一些参数，接收了这些参数后，该函数并不会立刻求值，而是继续返回另外一个函数，刚才传入的参数在函数形成的闭包中被保存起来。待到函数被真正需要求值的时候，之前传入的所有参数都会被一次性用于求值
```
var currying = function(fn) {
  var args = [];
  return function () {
    if(arguments.length === 0) {
      return fn.apply(this, args)
    } else {
      [].push.apply(args, arguments);
      return arguments.callee;
    }
  }
}

var cost = (function() {
  var money = 0;
  return function() {
    for(var i = 0, l = arguments.length; i < l; i++) {
      money += arguments[i];
    }
    return money;
  }
})()

cost = currying(cost); // 转换成currying函数

console.log(cost(100));
console.log(cost(200));
console.log(cost(300));
console.log(cost());
```
2. uncurrying
   - d 
3. 函数节流
   - 函数被频繁调用的场景
      - window.onresize事件，如果在window.onresize事件里有一些DOM操作，DOM节点操作是非常耗性能的，浏览器可能会出现卡顿现象
      - mousemove事件，如果给节点绑定了拖拽事件(主要是mousemove)，当节点被拖动的时候，会频繁触发拖拽事件函数
   - 函数节流的原理
      > 函数被触发的频率太高，按时间段来忽略一些事件请求，使用setTimeout实现
   - 函数接口的代码实现
      - throttle的原理：即将被执行的函数用setTimeout延迟一段时间执行，如果该次延迟执行还没有完成，则忽略接下来调用该函数的请求。
      ```
      var throttle = function(fn, interval) {
        var _self = fn, // 保存需要被延迟执行的函数的引用
            timer, // 定时器
            firstTime = true; //是否是第一次调用
        return function() {
          var args = arguments,
              _me = this;
          if(firstTime) { // 如果是第一次调用，不需要延迟
            _self.apply(_me, args);
            return firstTime = false;
          }
          if(timer) { // 如果定时器还在，说明前一次延迟执行还没有完成
            return false;
          }
          timer = setTimeout(function() { // 延迟一段时间执行
            clearTimeout(timer);
            timer = null;
            _self.apply(_me, args);
          }, interval || 500)
        }
      };
      ```
4. 分时函数
   - qq好友列表有成千上万个，让创建节点的工作分批进行
5. 惰性加载函数
   - 初级版本
    ```
    var addEvent = function(elem, type, handler) {
      if(window.addEventListener) {
        return elem.addEventListener(type, handler, false);
      }
      if(window.attachEvent) {
        return elem.attachEvent('on' + type, handler);
      }
    }
    ```

   - 惰性载入方案

    > addEvent被声明为一个普通函数，在函数里依然有分支判断。但是在第一次进入条件分支后，在函数内部会重写这个函数，重写之后的函数就是期望的addEvent函数，在下次进入addEvent函数的时候，addEvent函数里不再存在条件分支语句。

    ```
    var addEvent = function(elem, type, handler) {
      if(window.addEventListener) {
        addEvent = function(elem, type, handler) {
          elem.addEventListener(type, handler, false);
        }
      }
      if(window.attachEvent) {
        addEvent = function(elem, type, handler) {
          elem.attachEvent('on' + type, handler)
        }
      }
      addEvent(elem, type, handler)
    }
    ```
    
