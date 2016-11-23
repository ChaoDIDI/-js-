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
