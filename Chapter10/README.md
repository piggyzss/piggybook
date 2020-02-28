# 第十章 设计模式

<!-- toc -->

- [1、单例模式](#1、单例模式)
- [2、发布订阅模式（观察者模式）](#2、发布订阅模式（观察者模式）)
- [3、工厂模式](#3、工厂模式)
- [4、生产者消费者模式](#4、生产者消费者模式)
  <!-- tocstop -->


## 1、单例模式

#### **借助闭包实现单例模式**

``` javascript
// 代理实现单例模式
var ProxyMode = (function() {
  var instance = null;
  return function(name) {
    if(!instance) {
      instance = new CreateUser(name);
    }
    return instance;
  }
})();
// 测试单体模式的实例
var a = ProxyMode("aaa");
var b = ProxyMode("bbb");
// 因为单体模式是只实例化一次，所以下面的实例是相等的
console.log(a === b);    //true
```



## 2、发布订阅模式（观察者模式）

- **发布订阅模式**

**订阅者把自己想订阅的事件注册到调度中心，当发布者发布该事件也就是该事件触发时，由调度中心统一调度订阅者注册到调度中心的处理代码。**

在发布订阅模式里，发布者，并不会直接通知订阅者，换句话说，发布者和订阅者，彼此互不相识。

互不相识？那他们之间如何交流？答案是消息队列



**步骤：**

- 创建一个对象，这个对象用来存储一个缓存列表（调度中心）
- listen方法：用来给订阅者提供事件注册，其实就是把回调函数 fn 填加到对应event类型的缓存列表中
- trigger方法：用来给发布者发布事件到调度中心，调度中心处理代码，其实就是取到 arguments 里第一个参数当做 event类型，根据 event 类型去执行对应缓存列表中的回调函数
- remove方法可以根据 event 值取消订阅（取消订阅），其实就是把对应的事件从缓存列表中移除



**简单实现：**

``` javascript
var event = {
  clientList: {},
  listen: function (key, fn) {
      if(!this.clientList[key]) {
          this.clientList[key] = []
      }
      this.clientList[key].push(fn)
  }, 
  trigger: function () {
      var key = [...arguments].shift()
      var fns = this.clientList[key]
      if(!fns || fns.length === 0) {
        return false
      }
      for(let i=0, fn; fn = fns[i++];) {
          fn.apply(this, [...arguments])
      }
  },
  
  remove: function (key, fn) {
    var fns = this.clientList[key]
    if(!fns) {
      return false
    }
    if(!fn) {
      fns && (fns.length = 0)
    } else {
      for(let i=0; i < fns.length; i++) {
        if(fn == fns[i]) {
          fns.splice(i, 1)
        }
      }
    }
  }
}
```



- **发布订阅模式与观察者模式的对比**

> 《Learning JavaScript Design Patterns》一书这样说:
>
> “While the Observer pattern is useful to be aware of, quite often in the JavaScript world, we’ll find it commonly implemented using a variation known as the Publish/Subscribe pattern.”

**虽然 Observer 模式非常有用，但是在 JavaScript 的世界中，它更多的以一种被称为发布/订阅模式的变种来实现。**



**观察者模式**：观察者（Observer）直接订阅（Subscribe）主题（Subject），而当主题被激活的时候，会触发（Fire Event）观察者里的事件。

**发布订阅模式**：订阅者（Subscriber）把自己想订阅的事件注册（Subscribe）到调度中心（Event Channel），当发布者（Publisher）发布该事件（Publish Event）到调度中心，也就是该事件触发时，由调度中心统一调度（Fire Event）订阅者注册到调度中心的处理代码。

**差异**：

- 在观察者模式中，观察者是知道 Subject 的，Subject 一直保持对观察者进行记录。然而，在发布订阅模式中，发布者和订阅者不知道对方的存在。它们只有通过消息代理进行通信。
- 观察者模式的耦合性要强于发布订阅模式。
- 观察者模式大多数时候是同步的，比如当事件触发，Subject 就会去调用观察者的方法。而发布-订阅模式大多数时候是异步的（使用消息队列）。
- 观察者模式需要在单个应用程序地址空间中实现，而发布-订阅更像交叉应用模式。



**应用场景：**

- 可以广泛应用于异步编程中，替代回调函数
- 一个对象不用再显式的调用另一个对象的接口



## 3、工厂模式

**工厂模式是指提供一个创建对象的接口而不暴露具体的创建逻辑，可以根据输入类型创建对象。**

不暴露创建对象的具体逻辑，而是将将逻辑封装在一个函数中，那么这个函数就可以被视为一个工厂。

工厂模式根据抽象程度的不同可以分为：简单工厂、工厂方法和抽象工厂。

- 简单工厂模式

又叫静态工厂模式，由一个工厂对象决定创建某一种产品对象类的实例。主要用来创建同一类对象。

``` javascript
let UserFactory = function (role) {
  function User(opt) {
    this.name = opt.name;
    this.viewPage = opt.viewPage;
  }

  switch (role) {
    case 'superAdmin':
      return new User({ name: '超级管理员', viewPage: ['首页', '通讯录', '发现页', '应用数据', '权限管理'] });
      break;
    case 'admin':
      return new User({ name: '管理员', viewPage: ['首页', '通讯录', '发现页', '应用数据'] });
      break;
    case 'user':
      return new User({ name: '普通用户', viewPage: ['首页', '发现页'] });
      break;
    default:
      throw new Error('参数错误, 可选参数:superAdmin、admin、user')
  }
}
```

简单工厂的优点在于，你只需要一个正确的参数，就可以获取到你所需要的对象，而无需知道其创建的具体细节。但是在函数内包含了所有对象的创建逻辑（构造函数）和判断逻辑的代码，每增加新的构造函数还需要修改判断逻辑代码。

简单工厂只能作用于创建的对象数量较少，对象的创建逻辑不复杂时使用。



- 工厂方法模式

工厂方法模式的本意是将实际创建对象的工作推迟到子类中，这样核心类就变成了抽象类。

```javascript
// 安全模式创建工厂类
var Ball = function (type,name) {
// 判断实例是否属于ball
    if(this instanceof Ball) {
        var s = new this[type](name);
        console.log(s)
        return s;
    }else {
        return;
    }
}
// 工厂原型中设置创建所有类型数据对象的基类
Ball.prototype = {
    baoma: function(name) {
        this.play = function() {
            console.log('我在生产'+name);
        }
    },
    aodi: function(name) {
        this.play = function() {
            console.log('我在建造'+name);
        }
    },
}
// 客户需求
var baoma = new Ball('baoma','宝马');
// 开始建造
baoma.play();
// 客户需求
var aodi = new Ball('aodi','奥迪');
// 开始建造
aodi.play();
/*
    baoma {play: ƒ}
    我在建造宝马
    aodi {play: ƒ}
    我在生产奥迪
*/
```

工厂方法其实就是在工厂里面去写方法，在外部写一个公用的方法去调取工厂的独有方法，来实现客户的需求。



**应用场景：**

- 对象的构建十分复杂
- 需要依赖具体环境创建不同实例
- 处理大量具有相同属性的小对象



## 4、生产者消费者模式

