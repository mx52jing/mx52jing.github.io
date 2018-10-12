---
title: JS函数防抖、节流
date: 2018-10-11 16:03:09
categories:
- JavaScript
tags:
- JavaScript
---

在前端开发中会遇到一些频繁的事件触发，而事件频繁触发也会耗费性能，有可能导致页面卡顿或者浏览器崩溃,而函数防抖和节流可以帮助我们优化这些东西。

<!-- more -->

以一个搜索框为🌰：

当我们在搜索框内输入内容的话可能需要实现实时搜索功能，但是当我们输入很多文字时，就会发起很多次请求，这样显然是不太合适的。

🌰：

html代码

```

  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="utf-8">
    <title>JS Bin</title>
  </head>
  <body>
    <div>
      没有处理： <input type="text" id="normal">
    </div>
    <div>
      防抖：<input type="text" id="debounce">
    </div>
    <div>
      节流：<input type="text" id="throttle">
    </div>
  </body>
  </html>

```

js代码：

```
  function getElById(id){
    return document.getElementById(id)
  }

  const normalEl = getElById('normal')
  const debounceEl = getElById('debounce')
  const throttleEL = getElById('throttle')

  normalEl.addEventListener('keyup', function(){
    console.log('normal')
  })

```

### 函数防抖(debounce)

#### 原理：

在事件频繁触发的时候，我们规定一个时间间隔，如果下次触发事件时间-上次触发事件时间<设置的间隔时间，就忽略，不执行事件，当上次触发事件的时间和下次触发时间间隔大于设置的时间间隔，则再次触发事件。这样就能减少事件触发次数

#### 实现：

```
  // 接受两个参数，要执行的函数fn和间隔时间delay

  function debounce(fn, delay=500){
    let timer;
    //返回一个函数
    return function(){
      let _this = this, 
          args = arguments // 获取传入函数内的参数，触发事件时浏览器默认会传一个event参数，如果调用函数fn时不传这个参数，fn内将得不到event这个默认参数
      if(timer) clearTimeout(timer)
      timer = setTimeout(function(){
        fn.apply(_this, args)
      }, delay)
    }
  }


  debounceEl.addEventListener('keyup', debounce(function(e){
    console.log('debounce')
  }))

```

上面的方法是输入之后没有立即执行，还有一种情况是我希望立刻执行函数，然后等到停止触发 n 秒后，才可以重新触发执行。

我们可以添加一个参数，来控制是否立即执行

直接写两种结合版本吧

```

  function debounce(fn, immediate, delay=500){
    let timer;
    return function(){
      let _this = this,
          args = arguments
      if(timer) clearTimeout(timer)
      if(immediate) {
        if(!timer){
          fn.apply(_this, args)
        }
        timer = setTimeout(function(){
          timer = null
        }, delay)
      } else {
        timer = setTimeout(function(){
          fn.apply(_this, args)
        }, delay)
      }
    }
  }

```

### 函数节流(throttle)

#### 原理

无论你怎么频繁触发，我只在设置的时间段内执行一次事件，比如设置的时间间隔时500ms，那不管你500ms内触发多少次，我只执行一次，不管触发的有多频繁，我只是每隔500ms执行一次

#### 实现

第一种： 不会立即执行，每隔delay ms执行一次，停止触发后会再执行一次（无头有尾）

```

  function throttle(fn, delay = 500) {
    let timer;
    return function(){
      let _this = this,
          args = arguments
      if(timer) return
      timer = setTimeout(function(){
        fn.apply(_this, args)
        timer = null
      },delay)
    }
  }


  throttleEL.addEventListener('keyup',throttle(function(e){
    console.log('throttle')
  }))

```

第二种：立即执行，然后每隔delay ms执行一次,停止触发后不会再执行（有头无尾）

```

  function throttle(fn) {
    let prev = 0
    return function(){
      let _this = this,
        args = arguments,
        nowTime = Date.now()
      if(nowTime - prev > delay){
        fn.apply(_this, args)
        prev = nowTime
      }
    }
  }

```

第三种：

前面两种一个是不会立即触发，然后停止触发之后会再执行一次，一种是会立即触发，执行完之后不会再触发一次
那么如果想要立即触发并且停止触发之后会再执行一次，要怎么做呢：

代码：

``` 

  function throttle(fn, delay= 500) {
    let timer, prevTime = 0;
    return function() {
      let _this = this,
        args = arguments,
        nowTime = Date.now(),
        residualTime = nowTime - prevTime - delay
      if(residualTime > 0) {
        if(timer) {
          clearTimeout(timer)
          timer = null
        }
        fn.apply(_this, args)
        prevTime = nowTime
      } else {
        if(!timer) {
          timer = setTimeout(function(){
            fn.apply(_this, args)
            prevTime = nowTime
            timer = null
          }, delay)
        }
      }
    }
  }

```

### 不同

个人理解：

防抖的话如果在设置时间间隔内一直触发事件，就一直不执行，除非停止触发或者在大于时间间隔的时间段后触发才会执行。

节流的话事件是一直执行的，但是执行的频率大大减少了，不管你出发的有多频繁，只会在规定时间间隔内触发此事件



