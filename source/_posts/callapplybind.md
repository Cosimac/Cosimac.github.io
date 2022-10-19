---
title: call、apply、bind的区别与应用场景
date: 
tags: js
---
## call、apply、bind
### 为什么会有call和apply？
call和apply两个方法的作用基本相同，它们都是为了改变某个函数执行时的上下文（context）而建立的， 他的真正强大之处就是能够扩充函数赖以运行的作用域。通俗一点讲，就是改变函数体内部this 的指向。

``` javascript
window.color = "red";
var o = {color: "blue"};
function sayColor(){
	alert(this.color);
}
sayColor();//red
sayColor.call(this);//red，把函数体sayColor内部的this，绑到当前环境（作用域）(这段代码所处的环境)
sayColor.call(window);//red，把函数体sayColor内部的this，绑到window（全局作用域）
sayColor.call(o);//blue
```

### call( thisValue , arg1, arg2, ... )
> 注意：如果call方法没有参数，或者参数为null或undefined，则等同于指向全局对象。
``` javascript
window.color = "red";
var o = {color: "blue"};
function sayColor(){
	alert(this.color);
}
sayColor.call(this);//red
sayColor.call(window);//red
sayColor.call();
sayColor.call(null);
sayColor.call(undefined);
sayColor.call(o);//blue
```
### 应用场景
+ 判断对象类型
``` javascript
var arr = [];
Object.prototype.toString.call(arr); // [object Array]
//把函数体Object.prototype.toString()方法内部的this，绑到arr的执行环境（作用域）
```

### 手撕call
``` js
	var foo = {
	  count: 1
	};
	function bar() {
	  console.log(this.count);
	}
	bar.myCall(foo); // 1
--------------------------------------------------------------------
	Function.prototype.myCall = function(context) {
	  // 取得传入的对象（执行上下文），比如上文的foo对象，这里的context就相当于上文的foo
	  // 不传第一个参数，默认是window,
	  var context = context || window;
	  // 给context添加一个属性，这时的this指向调用myCall的函数，比如上文的bar函数
	  context.fn = this;//这里的context.fn就相当于上文的bar函数
	  // 通过展开运算符和解构赋值取出context后面的参数，上文的例子没有传入参数列表
	  var args = [...arguments].slice(1);
	  // 执行函数（相当于上文的bar(...args)）
	  var result = context.fn(...args);
	  // 删除函数
	  delete context.fn;
	  return result;
	};
```

### apply( thisValue , [arg1, arg2, ...] )
很明显，我们看标题的可以知道call和apply的一个区别了，它们两个唯一的区别就是传参列表的不同，apply是接收的参数是一个数组。

### 手撕apply

``` js
	var foo = {
	  count: 1
	};
	function bar() {
	  console.log(this.count);
	}
	bar.myApply(foo); // 1
--------------------------------------------------------------------
  Function.prototype.myApply = function(context) {
    var context = context || window;
    context.fn = this;
    var result;
    // 判断第二个参数是否存在，也就是context后面有没有一个数组
    // 如果存在，则需要展开第二个参数
    if (arguments[1]) {
      result = context.fn(...arguments[1]);
    } else {
      result = context.fn();
    }
    delete context.fn;
    return result;
  }
```

### 应用场景
+ 找出数组中最大或最小的元素

``` js
    var a = [10, 2, 4, 15, 9];
    Math.max.apply(Math, a); // 15
    Math.min.apply(null, a); // 2
    /* ES6的方法 */
    Math.max(...[10, 2, 4, 15, 9]); // 12 等同于Math.max(10, 2, 4, 15, 9);t
```

+ 可以将一个类似（伪）数组的对象（比如arguments对象）转为真正的数组。 前提： 被处理的对象必须有length属性，以及相对应的数字键。

``` js
  var ArrayLike = { // 一个类似数组的对象
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3
  }
  //接收的是对象，返回的是数组
  Array.prototype.slice.apply({0: 1, length: 1}) // [1]
  Array.prototype.slice.apply({0: 1}) // []
  Array.prototype.slice.apply({0: 1, length: 2}) // [1, undefined]
  Array.prototype.slice.apply({length: 1}) // [undefined]
  //（切下）[].slice(1, n)，返回索引为1到索引为n-1的数组

  /* ES6的方法 */
  Array.from(ArrayLike); // ["a", "b", "c"]

  // 没有部署Iterator接口的类似数组的对象，扩展运算符就“无法”将其转为真正的数组，但Array.from()可以
  ...ArrayLike; //  Found non-callable @@iterator
  // 只要是部署了Iterator接口的数据结构，Array.from 都能将其转为数组
```

+ 数组追加

``` js
var arr1 = [1,2,3];
var arr2 = [4,5,6];
[].push.apply(arr1, arr2);
console.log(arr1); // [1, 2, 3, 4, 5, 6]
console.log(arr2); // [4, 5, 6]
/* ES6的方法 */
arr1.push(...arr2); // [1, 2, 3, 4, 5, 6]
```

+ 数组合并

``` js
var arr1 = [1, 2, { id: 1, id: 2 }, [1, 2]];
var arr2 = ['ds', 1, 9, { name: 'jack' }];
// var arr = arr1.concat(arr2);//简单做法
Array.prototype.push.apply(arr1,arr2)
console.log(arr1);
/* ES6的方法 */
[...arr1,...arr2]
```

### bind( thisArg[, arg1[, arg2[, ...]]])
+ call和apply它们两个是改变this的指向之后立即调用该函数，而bind则不同，它是创建一个新函数，我们必须手动去调用它。
+ bind()是ES5新增的一个方法
+ 传参和call或apply类似
+ 不会执行对应的函数，call或apply会自动执行对应的函数
+ bind会返回对函数的引用

### 举个栗子

``` js
  var a ={
      name : "Cherry",
      fn : function (a,b) {
          console.log( a + b)
      }
  }
  var b = a.fn;
  b.call(a,1,2);//立即调用该函数
  b.bind(a,1,2)();//手动调用()，它返回一个原函数的拷贝（新的，不是原函数），并拥有指定的this值和初始参数。
```

### 应用场景

``` js
  var obj = {
    name:'JuctTr',
    age: 18
  };
  /**
   * 给document添加click事件监听，并绑定ExecuteFun函数
   * 通过bind方法设置ExecuteFun的this为obj
   */
  //document.addEventListener('click',ExecuteFun.call(obj),false);
  document.addEventListener('click',ExecuteFun.bind(obj),false);		
  function ExecuteFun(a,b){
      console.log(this.name, this.age);
  }
```

### 手撕bind

``` js
Function.prototype.myBind = function (context) {
  if (typeof this !== 'function') {
    throw new TypeError('Error')
  }
  var _this = this
  var args = [...arguments].slice(1)
  // 返回一个函数
  return function F() {
    // 因为返回了一个函数，我们可以 new F()，所以需要判断
    if (this instanceof F) {
      return new _this(...args, ...arguments)
    }
    return _this.apply(context, args.concat(...arguments))
  }
}
```
