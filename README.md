# -如何监听Js变量的变化-
 流行的MVVM的Js库/框架都有共同的特点就是数据绑定,在数据变更后响应式的自动进行相关计算并变更DOM 
 展现.所以可以理解为如何实现MVVM库/框架的数据绑定 
 常见的数据绑定的实现有脏值检查,基于ES5的getter和setter,以及ES已被飞起的Object.observe,和ES6中添加的Proxy.
 
 # 脏值检测
 angular使用的就是脏值检测,原始就是比较新值和旧值,当值真的发生变化时再更改DOM, 
 所以angular中又给$digest.那么为什么在想ng-click这样的内置指令在触发后会自动变更? 
 原理很简单, 在ng-click这样的内置指令中最后追加了$digest.
 简单的实现一个脏值检测:
 ```html
 <html>
 <head>
  <meta charser='urt-8'>
  <title>two-way bindingM</title>
 </head>
 <body onload='init()'>
 <button ng-click='inc'>Increase</button>
 <button ng-click='reset'>Reset</button>
 <span style='color:red' ng-lind='counter'></span>
 <span style='color:blue' ng-lind='counter'></span>
 <span style='color:green' ng-lind='counter'></span>
 ```
 
 
 ```javascript 
 <script text='javascript'>
 var counter = 0;
 function inc(){
  counter++;
 }
 function reset(){
 counter =0;
 }
 /* 数据模型区结束 
  * 绑定关系区开始********/
  function init(){
    bind();
  }
  function init(){
    var list =document.querySelectorAll('[ng-lick]');
    fot(var i=0; i<list.length;i++){
    list[i].onclick = (function(index)){
    return function(){
    window[list[index].getAttribute('ng-click')]();
    apply();
    };
    })(i);
    }
  }
  
  function apply(){
   var list =document.querySeleCtorAll("[ng-bind='counter']");
   for(var i=0;i <list.length;i++){
   if(list[i].innerHTML !=counter){
   list[i]innerHTML =counter;   }
   }
  }
  //绑定关系区结束
```
 </script>
 </body>
 </html>
 
 这样做的坏处是自己变更数据后,是无法自动改变DOM的,必须要想办法触发apply(),所以只能借助ng-click的包装,在ng-click中包含真是的click事件监听并追加 
 脏值检测以判断是否要更新DOM. 
 另外一个坏处是如果不注意, 每次脏值检测会检测大量的数据,而很多数据是没有检测的必要的,容易影响性能. 
  
  关于如何实现一个和angular一样脏值检测,知道原理后还有很多工作要去做,以及如何优化等等.



# ES5的getter与setter 
在ES5中新增了一个Object.defineProperty，直接在一个对象上定义一个新属性，或者修改一个已经存在的属性， 并返回这个对象。 
```javascript
 Object.defineProperty(obj, prop, descriptor)
``` 
 其接受的第三个参数可以取get和set并各自对应一个getter和setter方法：
 ```javascript
 var a = { zhihu:0 };

Object.defineProperty(a, 'zhihu', {
  get: function() {
    console.log('get：' + zhihu);
    return zhihu;
  },
  set: function(value) {
    zhihu = value;
    console.log('set:' + zhihu);
  }
});

a.zhihu = 2; // set:2
console.log(a.zhihu); // get：2
                      // 2
 
 ```
 基于ES5的getter和setter可以说几乎完美符合了要求。为什么要说几乎呢？ 
 
 首先IE8及更低版本IE是无法使用的，而且这个特性是没有polyfill的，无法在不支持的平台实现，
 这也是基于ES5getter和setter的Vue.js不支持IE8及更低版本IE的原因。也许有人会提到avalon，avalon在低版本IE借助vbscript一些黑魔法实现了类似的功能。

 除此之外，还有一个问题就是修改数组的length，直接用索引设置元素如items[0] = {}，以及数组的push等变异方法是无法触发setter的。 
 如果想要解决这个问题可以参考Vue的做法，在Vue的observer/array.js中，Vue直接修改了数组的原型方法：

```javascript
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

/**
 * Intercept mutating methods and emit events
 */

;[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
.forEach(function (method) {
  // cache original method
  var original = arrayProto[method]
  def(arrayMethods, method, function mutator () {
    // avoid leaking arguments:
    // http://jsperf.com/closure-with-arguments
    var i = arguments.length
    var args = new Array(i)
    while (i--) {
      args[i] = arguments[i]
    }
    var result = original.apply(this, args)
    var ob = this.__ob__
    var inserted
    switch (method) {
      case 'push':
        inserted = args
        break
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // notify change
    ob.dep.notify()
    return result
  })
})
```
 这样重写了原型方法，在执行数组变异方法后依然能够触发视图的更新。

 但是这样还是不能解决修改数组的length和直接用索引设置元素如items[0] = {}的问题，想要解决依然可以参考Vue的做法：
 前一个问题可以直接用新的数组代替旧的数组；后一个问题可以为数组拓展一个$set方法，在执行修改后顺便触发视图的更新。


 
