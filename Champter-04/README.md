#### Champter-04 函数
###### 递归
> 实现递归的两个条件：
> 1. 引用自身
> 2. 有终止条件

1. 普通函数实现递归
```js
/**
 * [判断一个字符串是否是回文]
 * @param  {[String]}  text [输入字符串]
 * @return {Boolean}
 */
  function isPalindrome(text){
    if(text.length <= 1){return true}
    if(text.charAt(0) != text.charAt(text.length - 1)){return false}
    return isPalindrome(text.substr(1, text.length - 2));
  }
```
2. 方法实现递归
```js
var ninja = {
  chirp: function(n){
    return n > 1 ? ninja.chirp(n - 1) + "-chirp" : "chirp";
  }
};
Assert(ninja.chirp(3) == "chirp-chirp=chirp", "OK");
```
2中方法实现递归存在的问题：引用丢失，如下
```js
var samurai = {chirp: ninja.chirp};
ninja = {};
try{
  assert(samurai.chirp(3) == "chirp-chirp-chirp", "Still work?");
}catch(e){
  assert(false, "Uh, this isn't work.");
}
```
可以不显示通过ninja调用chirp，而是通过函数上下文this引用
```js
  var ninja = {
    chirp: function(n){
      return n > 1 ? this.chirp(n - 1) + "-chirp" : "chirp";
    }
  };
```
3. 内联命名函数实现递归
```js
  var ninja = {
    chirp: function signal(n){
      return n > 1 ? signal(n - 1) + "-chirp" : "chirp";
    }
  }
```
重要一点：内联命名函数只能在自身函数内部可见，外部是不可见的
4. callee属性实现递归
ECMAScript 5在严格模式下已经禁止使用
```js
  var ninja = {
    chirp: function(n){
      return n > 1 ? arguments.callee(n - 1) + "-chirp" : "chirp";
    }
  };
```

###### 将函数视为对象
1. 函数赋值给变量
```js
  var fn = function(){};
```
2. 函数添加属性
```js
  var fn = function() {};
  fn.prop = "visible";
```
3. 函数存储（回调函数）  
通过给函数添加id属性标识函数唯一性
```js
  var store = {
    nextId: 1,

    cache: {},

    add :function(fn){
      if(!fn.id){
        fn.id = nextId ++;
        return !!(store.cache[fn.id] = fn);
      }
    }
  };
  function ninja(){}

  assert(store.add(ninja), "Function add");
  assert(!store.add(ninja), "Function has added");
```
4. 自记忆函数  
计算素数简单算法
```js
  function isPrime(value){
    if(!isPrime.answers) isPrime.answers = {};
    if(isPrime.answers[value] != null){
      return isPrime.answers[value];
    }
    var prime = value != 1;
    for(var i = 2; i < value; i++){
      if(value % i === 0){
        prime = false;
        break;
      }
    }
    return isPrime.answers[value] = prime;
  }
  assert(isPrime(5), "5 is prime");
  assert(isPrime.answers[5], "Cached!");
```
缓存记忆DOM元素
```js
  function getElements(name){
    if(!getElements.cache) getElements.cache = {};
    return getElements.cache[name]  = getElements.cache[name] || document.getElementsByTagNames(name);
  }
```
伪造数组方法
```js
  // <input type="text" id="first"/>
  // <input type="text" id="second"/>
/**
 * [把对象也可以作为数组使用]
 */
  var elems = {
    length: 0,

    add: function(ele) {
      Array.prototype.push.call(this, ele);
    },

    gather: function(name) {
      this.add(document.getElementById(name));
    }
  };
  elems.gather("first");
  assert(elems.length == 1 && elems[0].nodeType, "Verify an element in the object");
  elems.gather("second");
  assert(elems.length == 2 && elems[1].nodeType, "Add another one");
```
###### 函数重载
函数的length属性：  
> 通过lenght属性，可以知道声明了多少命名参数  
> 通过arguments.length可以知道调用时传入了多少参数

函数重载方法移步：[函数重载](https://github.com/xlshen/blog/blob/master/JavaScript函数重载（JavaScript%20Method%20Overloading）.md)
