### 实例化和原型
#### 原型链
详情：[JavaScript原型链](https://github.com/xlshen/JavaScript/issues/2)
#### HTML DOM原型
所有DOM元素都继承于HTMLELement构造器，通过访问HTMLElement的原型，浏览器可以实现扩展任意HTML节点的能力
```js
  HTMLElement.prototype.remove = function(){
    if(this.parentNode){
      this.parentNode.removeChild(this);
    }
  };
  var el = document.getElementById("elem");
  el.parentNode.removeChild(el);
  // 等价于
  document.getElementById("elem").remove();
```
#### 子例化原生对象
```js
  function MyArray(){}
  MyArray.prototype = new Array();
  var mine = new MyArray();
  mine.push(1, 2, 3);
  assert(mine.length === 3, "mine's length is 3");
```
但在IE中实现会出现问题；最好的解决策略是单独实现原生对象的各个功能，而不是扩展出子类。
```js
  function MyArray(){}
  MyArray.prototype.length = 0;
  (function(){
    var methods = ["push", "pop", "shift", "unshift", "slice", "splice", "join"];
    for(var i = 0; i < methods.length; i++){
      (function(name){
        MyArray.prototype[name] = function(){
          return Array.prototype[name].apply(this, arguments);
        }
      })(methods[i]);
    }
  })(methods[i]);
  var mine = new MyArray();
  mine.push(1, 2, 3);
  asssert(mine.length == 3, "Length = 3");
```
#### 实例化问题
```js
  function User(first, last){
    if(!this instanceof arguments.callee){
      return new User(first, last);
    }
    this.name = first + " " + last;
  }
```
#### 编程风格的代码
```js
  (function() {
    var initializing = false,
      superPattern = /xyz/.test(function() {
        xyz;
      })
        ? /\b_super\b/
        : /.*/;
    Object.subClass = function(properties) {
      var _super = this.prototype;

      initializing = true;
      var proto = new this();
      initializing = false;

      for (var name in properties) {
        proto[name] =
          typeof properties[name] == "function" &&
          typeof _super[name] == "function" &&
          superPattern.test(properties[name])
            ? (function(name, fn) {
                return function() {
                  var tmp = this._super;

                  this._super = _super[name];

                  var ret = fn.apply(this, arguments);
                  this._super = tmp;

                  return ret;
                };
              })(name, properties[name])
            : properties[name];
      }

      function Class() {
        if (!initializing && this.init) {
          this.init.apply(this, arguments);
        }
      }

      Class.prototype = proto;

      Class.constructor = Class;

      Class.subClass = arguments.callee;

      return Class;
    };
    })();

    var Person = Object.subClass({
    init: function(isDancing){
      this.dancing = isDancing;
    },
    dance: function(){
      return this.dancing;
    }
    });

    var ninja = Person.subClass({
    init: function(){
      this._super(false);
    },
    dance: function(){
      return this._super();
    },
    swing: function(){
      return true;
    }
  });
```
