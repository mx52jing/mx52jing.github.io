---
title: JS观察者模式
date: 2018-09-20 22:34:20
categories:
- JavaScript
tags:
- JavaScript
---

### 观察者模式

观察者模式又叫发布订阅模式（Publish/Subscribe），它定义了一种一对多的关系，让多个观察者对象同时监听某一个主题对象，这个主题对象的状态发生变化时就会通知所有的观察者对象，使得它们能够自动更新自己。

<!-- more -->


当我们做一件事情时，想要通知某个人做相对应的事情，这时就可以使用观察者模式。

观察者模式可以支持简单的广播通信，自动通知订阅过的对象。

可以降低部分代码的耦合性


下面是我自己的实现方式

```
  // 提供3个API，分别是订阅、发布、移除

  const Events = (function(){

    // 定义一个对象，对象内是要订阅的事件集合

    const eventList = {}

    // 订阅

    function listen(name, fn){

      // 如果没有这个key，就创建一个

      if(!eventList[name]) {
        eventList[name] = []
      }
      eventList[name].push(fn)

      // 返回this可以链式调用
      return this
    }

    // 发布

    function trigger() {

      // 取出第一个参数，也就是要发布的事件名

      const name = [].shift.call(arguments)

      if(!eventList[name] || !eventList[name].length) return false

      // 遍历事件数组，调用函数

      eventList[name].forEach(item => {
        item.apply(this, arguments)
      })

      // 返回this可以链式调用

      return this
    }

   // 移除

   function remove(name, deletefn) {

    // 如果没有这个名称，就直接返回

    if(!eventList[name]) return false

    // 如果没有制定某个函数，就将这个键名内的函数全部删除

    if(!deletefn) {
      eventList[name] && (eventList[name] = [])
      return
    }

    eventList[name].forEach((item, index) => {
      if(item == deletefn) {
        eventList[name].splice(index, 1)
      }
    })

    // 返回this可以链式调用

    return this
   }  

    return {listen,trigger, remove}

  })()

```

使用方法

```

  Events.listen('app',function(){
    console.log('app')
  })

  Events.trigger('app') // 打印出 app

  Events.remove('app')

```

或者

```

  Events.listen('app',function(){
    console.log('app')
  }).trigger('app').remove('app')

```

以上就是观察者模式的大致原理了，写得还是比较简单的，有很多方面可能还没考虑到

**不足之处，请多指教**