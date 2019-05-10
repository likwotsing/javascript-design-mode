# 第4章 单例模式

- 单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点。
- 比如：线程池、全局缓存、浏览器中的window对象
- 用一个变量来标志当前是否已经为某个类创建过对象，如果是，则在下一次获取该类的实例时，直接返回之前创建的对象
- 核心：确保只有一个实例，并提供全局访问

```
单例模式是一种简单但非常实用的模式，特别是惰性单例技术，在合适的时候才创建对象，并且只创建唯一的一个。把创建对象和管理单例的职责分布在两个不同的函数中，组合起来使用更好。
```
  
## 降低全局变量带来的命名污染
1. 使用命名空间
   - 不会杜绝全局变量，但可以减少全局变量的数量
     ```
     var namespace = {
       a: function() {
         alert(1)
       },
       b: function() {
         alert(2)
       }
     }
     ```
   - 动态地创建命名空间
     ```
     var MyApp = {};
     MyApp.namespace = function(name) {
       var parts = name.split('.');
       var current = MyApp;
       for(var i in parts) {
         if(!current[parts[i]]) {
           current[parts[i]] = {};
         }
         current = current[parts[i]];
       }
     }
     MyApp.namespace('event');
     MyApp.namespace('dom.style');
     // 上述代码等价于
     var MyApp = {
       event: {},
       dom: {
         style: {}
       }
     }
     ```
2. 使用闭包封装私有变量
  ```
  var user = (function() {
    var _name = 'sven',
        _age = 18;      // 使用下划线来约定私有变量
    return {
      getUserInfo: function() {
        return _name + '-' + _age;
      }
    }
  })();
  ```
## 4.5 惰性单例

> 惰性单例是指在需要的时候才创建对象实例。比如登录框，总是唯一的。

1. 第一种解决方案：在页面加载完成的时候便创建登录的div浮窗，一开始是隐藏的，当用户点击登录按钮的时候，它才开始显示。如果用户不需要进行登录操作，就会白白浪费一些DOM节点。
2. 第二种解决方案：惰性单例，可以用一个变量判断是否已经创建过登录浮窗。
```
var createLoginLayer = (function() {
  var div;
  return function() {
    if(!div) {
      div = document.createElement('div');
      div.innerHTML = '我是登录浮窗';
      div.style.display = 'none';
      document.body.appendChild(div);
    }
    return div;
  }
})();

document.getElementById('loginBtn').onclick = function() {
  var loginLayer = createLoginLayer();
  loginLayer.style.display = 'block';
}
```

- 上面的惰性单例是违反单一职责原则的，创建对象和管理单例的逻辑都放在createLoginLayer内部
- 如果下次要创建唯一的iframe或script标签，那就需要把createLoginLaery函数几乎复制一遍（只是修改了标签）。

## 4.6 通用的惰性单例

```
var getSingle = function(fn) {
  var result;
  return function() {
    return result || (result = fn.apply(this, arguments));
  }
}

var createLoginLayer = getSingle(createIframe);

var createIframe = function() {
  var iframe = document.createElement('iframe');
  document.body.appendChild(iframe);
  return iframe;
}
```