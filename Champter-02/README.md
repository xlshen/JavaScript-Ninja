#### Champter-02 测试框架构建
###### 1. 断言
HTML:
```html
  <!DOCTYPE html>
  <html>
    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="X-UA-Compatible" content="ie=edge">
      <title>Assert</title>
      <style media="screen">
        /**
         * [3. 断言样式]
         */
        #results li.pass{color: green}
        #results li.fail{color: red}
      </style>
    </head>
    <body>
      <!-- [4. 显示断言结果] -->
      <ul id="results"></ul>
    </body>
  </html>
```
JavaScript:
```js
    /**
     * [1. 定义断言方法]
     * @param  {[Boolean]} value [断言条件]
     * @param  {[String]} desc  [断言描述]
     */
    function assert(value, desc){
      var li = document.createElement("li");
      li.className = value ? "pass" : "fail";
      li.appendChild(document.createTextNode(desc));
      document.getElementById("results").appendChild(li);
    }
    /**
     * [2. 执行断言]
     */
    window.onload = function() {
      assert(true, "The test suit is running.");
      assert(false, "Fail!");
    }
```
##### 2. 测试组
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Test Suit for Synchronous</title>
  <style media="screen">
    #results li.pass{color: green}
    #results li.fail{color: red}
  </style>
</head>
<body>
  <script type="text/javascript">
  (function() {
  var results;
  this.assert = function assert(value, desc) {
    var li = document.createElement("li");
    li.className = value ? "pass" : "fail";
    li.appendChild(document.createTextNode(desc));
    results.appendChild(li);
    if (!value) {
      li.parentNode.parentNode.className = "fail";
    }
    return li;
  };
  this.test = function test(name, fn) {
    results = document.getElementById("results");
    results = assert(true, name).appendChild(document.createElement("ul"));
    fn();
  };
})();

window.onload = function() {
  test("A test.", function() {
    assert(true, "First assertion completed");
    assert(true, "Second assertion completed");
    assert(true, "Third assertion completed");
  });
  test("Another test.", function() {
    assert(true, "First assertion completed");
    assert(false, "Second assertion failed");
    assert(true, "Third assertion completed");
  });
  test("A third test.", function() {
    assert(null, "fail");
    assert(5, "pass");
  });
};
</script>
  <ul id="results"></ul>
</body>
</html>
```
##### 3. 异步测试
> 1. 将依赖的相同异步操作的断言组合成一个统一的测试组
> 2. 每个测试组需要放在一个队列上，在先前其他的测试组完成执行之后再执行

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Test Suit for Asychronous</title>
  <style media="screen">
    #results li.pass{color: green}
    #results li.fail{color: red}
  </style>
</head>
<body>
  <script type="text/javascript">
  (function() {
  var queue = [],
    paused = false,
    results;
  this.test = function(name, fn) {
    queue.push(function() {
      results = document.getElementById("results");
      results = assert(true, name).appendChild(document.createElement("ul"));
      fn();
    });
    runTest();
  };
  this.pause = function() {
    paused = true;
  };
  this.resume = function() {
    paused = false;
    setTimeout(runTest, 1);
  };
  function runTest() {
    if (!paused && queue.length) {
      queue.shift()();
      if (!paused) {
        resume();
      }
    }
  }
  this.assert = function assert(value, desc) {
    var li = document.createElement("li");
    li.className = value ? "pass" : "fail";
    li.appendChild(document.createTextNode(desc));
    results.appendChild(li);
    if (!value) {
      li.parentNode.parentNode.className = "fail";
    }
    return li;
  };
})();

window.onload = function() {
  test("Async Test #1", function() {
    pause();
    setTimeout(function() {
      assert(true, "First test completed");
      resume();
    }, 1000);
  });

  test("Async Test #2", function() {
    pause();
    setTimeout(function() {
      assert(false, "Second test failed");
      resume();
    }, 1000);
  });
};
</script>
  <ul id="results"></ul>
</body>
</html>
```
