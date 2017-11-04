#### Champter-03 函数
###### 1. 函数声明
> 所有的函数都有一个name属性，该属性保存着该函数名称的字符串，没有名称的函数也仍然有name属性，只不过该属性值为空字符串。

```js
/**
 * [如果将函数赋值给变量，则变量的name属性的值是函数名称，而非变量本身的名称，如果赋值的是匿名函数，则为空字符串]
 */
window.wieldSword = function swingSword(){return true};
assert(window.wieldSword.name === "swingSword", "wieldSword's real name is swingSword!");
```
###### 2. 作用域和函数
1. 变量声明的作用域开始于声明的地方，结束于所在函数的结尾，与代码嵌套无关  
2. 命名函数的作用域是指该函数的整个函数范围，与代码嵌套无关  
3. 对于作用域声明，全局上下文相当于一个把所有代码包裹起来的超大型函数

###### 3. 函数调用
该部分内容详解传送门：[this的前世今生](https://github.com/xlshen/JavaScript/issues/1)
1. 作为函数调用  
2. 作为方法调用  
3. 作为构造器调用  
4. 通过apply()或call()调用  

ps：在回调函数中强制指定函数上下文
```js
/**
 * [forEach函数]
 */
function forEach(list, callback){
  for(var n = 0; n < list.length; n++){
    callback.call(list[n], n);
  }
}
var names = ["fn_1", "fn_2", "fn_3"];
forEach(names, function(index){
  assert(this == names[index], "Got the value of " + names[index]);
});
```
