### Champter-05 闭包
#### 闭包原理
> 闭包（closure）是一个函数在创建时允许该自身函数访问并操作该自身函数之外的变量时所创建的作用域。换句话说，闭包可以让函数访问所有变量和函数，只要这些变量和函数都存在于该函数声明时的作用域中就可以。

1. 内部函数的参数是包含在闭包中
2. 作用域之外的变量，即便是函数声明之后的变量，都存在于闭包中
3. 相同的作用域内，尚未声明的变量不能进行提前引用

#### 使用闭包
1. 私有变量  
> 通过闭包隐藏内部变量，只能通过内部控制方法进行修改和访问

```js
  function Ninja(){
    var feints = 0;

    this.getFeints = function(){
      return feints;
    };
    this.feint = function(){
      feints ++;
    };
  }

  var ninja = new Ninja();
  ninja.feint();
  assert(ninja.getFeints() == 1, "Access to feint");
  assert(ninja.feints === undefined, "Can't access to feint directly");
```
#### 回调和定时器
普通回调：`ajax`成功回调函数
定时器回调：
```js
  function animateIt(elementId){
    var elem = document.getElementById(elementId);
    var tick = 0;

    var timer = setInterval(function(){
      if(tick < 100){
        elem.style.left = elem.style.top = tick + "px";
        tick ++;
      }else{
        clearInterval(timer);
        assert(tick == 100, "Tick accessed via a closure");
        assert(elem, "Element accessed");
        assert(timer, "Timer accessed");
      }
    });
  }
```
#### 绑定函数上下文
```js
  var button = {
    clicked: false,
    click: function(){
      this.clicked = true;
      assert(button.clicked, "The button has been clicked");
    }
  };

  var elem = document.getElementById("test");
  elem.addEventListener("click", button.click, false);
```
浏览器事件处理对象默认是触发的元素，而不是上面的button；
但是我们可以通过`匿名函数、apply()和闭包`强制让特定的函数调用时使用特定的上下文环境。
```js
  function bind(context, name){
    return function(){
      return context[name].apply(context, arguments);
    }
  }
  // 上面代码，唯一不同的是，在调用时用bind函数传递事件处理
  elem.addEventListener("click", bind(button, "click"), false);
```
Prototype流行库中bind()如下：
```js
Function.prototype.bind = function(){
  var fn = this, args = Array.prototype.slice.call(arguments),
  object = args.shift();
  return function(){
    return fn.apply(object, args.concat(Array.prototype.slice.call(arguments)));
  }
};
var myObject = {};
function myFunction(){
  return this == myObject;
}

assert(!myFunction, "Context has not set");
var aFunction = myFunction.bind(myObject);
assert(aFunction(), "Context has set");
```
#### 偏应用函数
分部函数：上述绑定函数其实就是一个分部函数应用，但作为普通分部函数来说，我们可以构造更为通用的部分。
```js
  Function.prototype.partial = function(){
    var fn = this, args = Array.prototype.slice.call(arguments);
    return function(){
      var arg = 0;
      for(var i = 0; i < args.length && arg < arguments.length; i++){
        if(args[i] === undefined){
          args[i] = arguments[arg++];
        }
      }
      return fn.apply(this, args);
    }
  };
```
该函数实现了基本的`curry()`方法，但是不同的是用户可以指定输入参数的个数，可以在参数列表的位置指定所需要的参数，然后在后续调用中，根据空缺的参数是否等于`undefined`来判断填充。
#### 函数重载
我们可以在用户无感知的情况下操作一个函数的内部行为：1. 修改现有函数 2. 基于现在创建一个自更新函数
##### 缓存记忆
第四章中提到的缓存记忆方法是通过修改现有函数来实现缓存记忆功能，在此我们使用自更新
```js
  Function.prototype.memoized = function(key){
    this._value = this._values || {};
    return this._values(key)[key] !== undefined ? this._values[key] : this._values[key] = this.apply(this, arguments);
  };
  function isPrime(num){
    var prime = num != 1;
    for(var i = 2; i < num; i++){
      if(num % i == 0){
        prime = false;
        break;
      }
    }
    return prime;
  }
  assert(isPrime.memoized(5), "The function works; 5 is prime.");
  assert(isPrime._value[5], "The answer has been cached.");
```
该方法缺点是，isPrime()函数的调用者必须记住调用的memoized()方法才能使用缓存记忆功能。优化如下：
```js
  Function.prototype.memoized = function(key){
    this._value = this._values || {};
    return this._values(key)[key] !== undefined ? this._values[key] : this._values[key] = this.apply(this, arguments);
  };
  Function.prototype.memorize = function(){
    var fn = this;
    return function(){
      return fn.memoized.apply(fn, arguments);
    }
  };
  var isPrime = (function(num){
    var prime = num != 1;
    for(var i = 2; i < num; i++){
      if(num % i == 0){
        prime = false;
        break;
      }
    }
    return prime;
  }).memorize();
  assert(isPrime(17), "17 is prime");
  ```

#### 函数包装
用于单个步骤内重载创建新函数或继承函数，最有价值的场景，在重载一些已经存在的函数时，同时保持原始函数在包装后仍然能用。
```js
  function wrap(object, method, wrapper){
    var fn = object[method]; // 获取原始对象方法
    return object[method] = function(){
      return wrapper.apply(this, [fn.bind(object)].concat(Array.prototype.slice.call(arguments)));
    };
  }
  // 本案例重写了method方法，用original参数形式代替
  wrap(XX, "method", function(original, elem, attr){
    return attr == "title" ? elem.title : original(elem, attr);
  });
```
#### 即时函数
```js
(function(){})() === (var fn = function(){};(fn)());
// 1. 创建一个函数实例
// 2. 执行该函数
// 3. 销毁该函数
```
可以通过即时函数来创建临时作用域和私有变量，例如jQuery在全局中创建了一个名为jQuery的对象，还为该函数创建了一个$的别名。但是会存在第三方其他库别名也为$的问题，我们可以通过即时函数来解决此类问题。
```js
  $ = function(){alert("not jQuery")};
  (function($){
    // 内部还是通过$来引用jQuery
    $("XXX").method();
  })(jQuery);
```
#### 类库包装
另一个应用闭包和即时函数的地方就是JavaScript类库的开发。当开始类库时，很重要一点就是不希望一些不必要的变量去污染全局命名空间，尤其是那些临时变量。要解决这个问题，闭包和即时函数尤其有用。
```js
  (function(){
    var jQuery = window.jQuery = function(){
      //
    };
  })();
```
1. 外部通过全局变量jQuery来引用。
2. 为了避免全局jQuery变量被修改，内部创建局部变量jQuery在内部调用，强制将其保持在即时函数的作用域内。
```js
  // 该段代码和上述代码实现同样的效果，只是代码组织方式不同而已。如果只输出一个变量时，优先使用该技巧
  var jQuery = (function(){
    function jQuery(){
      //
    }
    return jQuery;
  })();
```
