### 正则表达式
#### 正则表达式表示
1. 字面量
2. 构造RegExp对象的实例
```js
  var pattern = /test/;
  var pattern = new RegExp("test");
```

开发时优先使用字面量形式，而构造器方式则是用于动态构建正则表达式。字面量优于字符串的其中一个原因是反斜杠在正则中非常重要，但是反斜杠在普通字符串中也是一个转义字符，所以，在字符串中要用双反斜杠`\\`表示，表现起来很怪异。
> 1. i——匹配不区分大小写，所以/test/i不仅可以匹配“test”，还可以是“TEST”，“Test”等
> 2. g——匹配所有实例，全局内匹配，而不是默认第一个实例之后就不进行匹配。要匹配所有可能的结果
> 3. m——允许匹配多行，比如可以匹配文本框（textarea）中内容

#### 术语和操作符
1. 精准匹配
```js
  var pattern = /test/; // 必须匹配含有“test”的字符串，t后面跟着e，e后面跟着s，s后面跟着t
```
2. 匹配一类字符  
字符类操作符：`[abc]`, `[^abc]`, `[a-z]`
```js
  var pattern = /[abc]/; // 匹配a, b, c中任何一个字符，即使这个表达式内容再多，也只能匹配其中一个！
  var pattern = /[^abc]/; // 插入符^，匹配除了a, b, c之外的任意字符。
  var pattern = /[a-f]/; // 匹配范围， 匹配a到f之间的任意字符（包含a和f），当然我们可以写成[abcdef]，但是上述写法更简便，更清晰
```
3. 转义字符：`\`  
如果我们需要输入特殊字符，比如：`.`，`$`这样的字符，因为它们在正则中有特殊的含义，我们需要用反斜杠`\`进行转义，`\.`、`\$`进行表示。
4. 开始和结束：`^`, `$`  
我们有时需要确保字符串的开始和结束需要特定的字符，比如手机号，开始和结束必须是数字，这需要我们用匹配开始和结束的符号`^`和`$`进行位置匹配。
```js
  var pattern = /^test$/; // 必须是以test开头，test结尾的字符，也就是匹配字符必须是test
```
5. 重复出现
> 1. ?: 在一个字符后面加一个问号，表示该字符是可选的，可以出现0次或者1次。例如：/t?est/可以匹配"test"和"est"
> 2. +: 在一个字符后面加一个加号，表示该字符出现至少一次，也就说1次或者多次。例如：/t+est/可以匹配"test","ttest","tttest"等
> 3. \*: 在一个字符后面加一个星号，表示该字符出现0次或者多次。例如：/t*est/可以匹配"est","test","ttest"等
> 4. {n, m}: 在一个字符后面加一个花括号，表示该字符出现固定n-m次数。例如：/test{4}匹配“testttt”，/test{1,3}/可以匹配"test","testt"和"testtt"，第二个参数是可选的，如果省略的话表示至少n次

  这些重复的操作符都是贪婪的（`greedy`）或者非贪婪的（`lazy`）。默认情况下都是贪婪的：它们匹配所有的字符组合。在操作符后面增加一个问号?字符表示非贪婪匹配：进行最小限度的匹配

6. 预定义字符类和字符术语    
`\t`: 水平制表符  
`\b`: 空格  
`\v`: 垂直制表符  
`\f`: 换页符  
`\r`: 回车  
`\n`: 换行符  
`. `: 匹配除了换行符(`\n`)之外的任意字符  
`\d`: 匹配任意数字，等价于`[0-9]`  
`\D`: 匹配任意非数字，等价于`[^0-9]`  
`\w`: 匹配包含下划线在内的所有单词字符，等价于`[0-9a-zA-Z_]`  
`\W`: 匹配任意非单词字符，等价于`[^0-9a-zA-Z]`  
`\s`: 匹配任意空白字符，包含空格，制表符，换页符等  
`\S`: 匹配任意非空白字符  
`\b`: 匹配单词的边界  
`\B`: 匹配非单词边界  
7. 分组  
当正则表达式有一部分使用括号进行分组时，它具有双重责任，同时也是创建了所谓的捕获（capture）。  

  或操作符：  
  可以用竖线（`|`）字符表示或者的关系。例如：`/a|b/`匹配"a"或"b"字符,`/(ab)+|(cd)+/`匹配出现一次或者多次的"ab"或者"cd"

  `反向引用`：
  这种术语表示法是在反斜杠后面加一个要引用的捕获数量，数字从1开始，如`\1`，`\2`等。举例来说，`/^([dtn])a\1/`可以匹配任意一个以“d”，“t”或“n”开头，后面紧跟一个“a”字符，并且后面跟着和第一个捕获相同字符的字符串。`\1`的匹配只有在执行时才能确定到底是哪个捕获组。  
  要在匹配HTML时也很有用，例如：`/<(\w+)>(.+)<\/\1>/`，可以匹配像`<strong>whatever</strong>`这样的元素
8. 编译正则表达式
```html
  <div class="samurai ninja"></div>
  <div class="ninja samurai"></div>
  <div></div>
  <span class="samurai ninja"></span>
  <script type="text/javascript">
    function findClassInElements(className, type){
      var elems = document.getElementByTagName(type || "*");
      var regex = new RegExp("(^|\\s)" + className + "(\\s|$)");
      var results = [];
      for(var i = 0, length = elems.length; i < length; i++){
        if(regex.test(elems[i].className)){
          results.push(elems[i]);
        }
      }
      return results;
    }
  </script>
```
9. 进行全局表达式匹配

  通过`String`的`match()`方法进行局部匹配时，返回的数组包含了匹配成功整个字符串和其他捕获结果。  
  但当应用全局正则表达式时，返回的东西不一样。返回值仍然是个数组，但是全局表达式下匹配所有可能的匹配结果，而不是第一个匹配结果，返回的数组包含了匹配的全局结果。这种情况下，单个匹配的结果时不会返回的。
  ```js
    var html = "<div class='test'><b>Hello</b> <i>world!</i></div>";
    var results = html.match(/<(\/?)(\w+)([^>]*?)>/);
    assert(results[0] == "<div class='test'>", "The entire match");
    assert(results[1] == "", "The missing slash");
    assert(results[2] == "div", "The tag name");
    assert(results[3] == " class='test'", "The attribute");

    var results = html.match(/<(\/?)(\w+)([^>]*?)>/g);
    assert(results[0] == "<div class='test'>", "Opening div tag");
    assert(results[1] == "<b>", "Opening b tag");
    assert(results[2] == "</b>", "Closing b tag");
    assert(results[3] == "<i>", "Opening i tag");
    assert(results[4] == "</i>", "Closing i tag");
    assert(results[5] == "</div>", "Closing div tag");
  ```
  如果捕获对我们来说很重要，我们可以使用正则表达式的`exec()`方法，在全局正则情况下，每调用一次，都会返回当前捕获组的匹配结果。
10. 捕获的引用  
  有两种方法，可以引用到捕获的结果：一个是自身匹配:`\1`，一个是替换字符串`replace()`
```js
  "fontFamily".replace(/(A-Z)/g, "-$1").toLowerCase(); // font-family
```
这种情况下，我们通过`$1`, `$2`等来引用替换的字符串
11. 没有捕获的分组  
  因为小括号不仅表示捕获，还表示分组。这种情况下就造成了不需要捕获的也会在捕获中显示，我们可以通过使用`被动子表达式`来标记那些组不需要捕获：`?:`
```js
  var pattern = /((?:ninja-)+)sword/; // 只有外层的才会被捕获
```
12. 利用函数进行替换  
`replace()`方法的强大之处是可以接受一个函数作为替换值，而不是一个固定的字符串。  
当作为函数为替换值时，该函数接受四个参数：`1. 匹配的完整文本` 2. `匹配的捕获，一个捕获对应一个参数` 3. `匹配字符在源字符串中的索引` 4. `源字符串`
```js
  var pattern = /(\w+)(-)/g;
   var result = "ninja-ninja-sword".replace(pattern, function(all, first, second){
     return first.toUpperCase() + "+";
   });
   alert(result);
```
两个捕获组，一个捕获组对应一个参数
13. 利用正则表达式解决常见问题  
  1. 修剪字符串  
  `String.trim()`方法不能兼容旧版本浏览器，我们通过正则实现：
  ```js
    // Method 1
    function trim(str){
      return (str || "").replace(/^\s+|\s+$/g, "")
    }
    // Method 2
    function trim(str){
      return str.replace(/^\s\s*/, "").replace(/\s\s*$/, "");
    }
    // Method 3
    function trim(str){
      var str = str.replace(/^\s\s*/, ""),
          ws = /\s/,
          i = str.length;
      while(ws.test(str.charAt(--i))); // 通过获取该位置有没有字符判断
      return str.slice(0, i + 1);
    }
  ```
以上三种方式在短字符串，前两种有明显优势，但在长字符串的文档中，第三种优势更大。
  2. 匹配换行符  
  点`.`匹配换行符以外的任意字符
  ```js
    var html = "<b>Hello</b>\n<i>world</i>";
    assert(/.*/.exec(html)[0] == "<b>Hello</b>", "没有匹配换行符");
    assert(/[\S\s]*/.exec(html)[0] == "<b>Hello</b>\n<i>world</i>", "匹配所有");
    assert(/(?:.|\s)*/.exec(html)[0] == "<b>Hello</b>\n<i>world</i>", "匹配所有");
  ```
  3. 匹配Unicode  
  创建一个包含整个Unicode字符集：`/\w\u0080-\uFFFF_-/`
