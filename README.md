# frame-design
数组化
-
浏览器下存在很多类数组对象，如function中的arguments，通过document.forms，document.getElementsByClassName等方式获取的节点集合
```javascript
  var arrayLike = {
    0: 'a',
    1: '1',
    2: '2',
    length: 3
  }
```
类数组的功能弱，所以使用时都会通过[].slice.call做一个转换。但旧IE的nodelist不是Object的子类，这时候只能通过遍历这个对象
```javascript
  //jQuery的makeArray
  var makeArray = function(array){
    var ret = [];
    if(array != null){
      var i = array.length;
      //如果是window对象等
      if(i == null || typeof array == 'string' || jQuery.isFunction(array) || array.setInterval){
        ret[0] = array;
      } else {
        while(i){
          ret[--i] = array[i];
        }
      }
      return ret;
    }
  }
```
无冲突处理
-
$是重要的函数名，很多库都会使用
```javascript
  var window = this,
      undefined,
     //如果还没引入jQuery为undefined
     _jQuery = window.jQuery,
     //把window.$保存下来
     _$ = window.$;
     //赋值是从右到左的
     jQuery = window.jQuery = window.$ = function(selector, context){
        return new jQuery.fn.init(selector, context);
     }
   jQuery.extend({
      noConflict: function(deep){
        window.$ = _$;
        if(deep)
          window.jQuery = _jQuery;
        return jQuery;
      }
   });
```
使用时，先引入别的库，然后引入jQuery, 使用$.noConflict()进行改名，这样就不影响别人的$运作了
```javascript
  var jq = $.noConflict();
```

模块加载系统
-
AMD是Asinchronous Module Definition的缩写，意为异步模块定义。主要接口有两个，define和require。define第一个参数为模块id,第二个为模块列表
第三个为工厂方法。
```javascript
  define('xxx', ['a', 'b'], function(a, b){
  });
```
require的参数第一个为依赖列表，第二个为回调
```javascript
  require('xxx', function(a, b){
  });
```
require方法的作用是等依赖列表都加载完毕，执行用户回调。加载过程分为以下几步：

 * 取得依赖列表的第一个ID，转换为URL
 * 检测依赖模块有没有被加载过，或者正在加载。
 * 创建script节点，绑定onerror、onload、onreadychange等回调，插入dom树
 * 将模块的依赖列表
 
 语言模块
 -
 语言模块比如字符串的扩展与修复。
 contains方法，用来判定一个字符串是否包含另一个字符串，用正则性能太差
 ```javascript
    function contains(target, it){
        return target.indexOf(it) != -1;
    }
 ```
 在mootools版本中，它支持更多参数，比如判断className中是否包含class，就可以使用`contains(target, class, ' ')`
 ```javascript
    function contains(target, str, separator){
        return separator ? 
              (separator + target + separator).indexOf(separator + str + separator) != -1 :
              target.indexOf(it) != -1;
    }
 ```
startsWith方法，判断目标字符串是否位于原字符串的开始处
```javascript
    function startsWith(target, str, ignorecase){
        //截取最开始的length字符串
        var start_str = target.substring(0, str.length);
        return ignorecase ? start_str.toLowerCase() === str.toLowerCase()：
          start_str === str;
    }
```
repeat方法，将一个字符串重复自身n次
```javascript
    function repeat(target, n){
        //创建一个n+1的空数组，直接join成字符串
        return (new Array(n + 1)).join(target)
    }
```
来看一下改良版，比如我们将'str'重复5次，第三次直接用'strstr'来操作
```javascript
    function repeat(target, n){
        var s = target, total = '';
        while(n > 0){
            if(n % 2 == 1){
               total += s;
            } 
            if(n == 1){
               break;
            }
            s += s;
        }
        //右移一位，相当于/2
        n = n >> 1;
    }
```
