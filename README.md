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
 
 下面看下语言模块的扩展与修复：
 
 字符串的扩展与修复
 -
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
byteLen方法，取得字符串所有字节的长度，如果将一个英文字符插入数据库char/varchar/text类型的字段占用一个字节，中文字符需要占用两个字节。
```javascript
    function byteLen(target){
        var byteLength = target.length,
            i = 0;
        for(; i < target.length; i++){
            if(target.charCodeAt(i) > 255){
                byteLength++;
            }
        }
        return byteLength;
    }
```
camelize方法：转换为驼峰风格（特定字符串替换使用正则）
```javascript
    function camelize(target){
        if(target.indexOf('-') < 0 && target.indexOf('_') < 0){
            return target;
        }
        return target.replace(/[-_][^-_]/g, function(match){
            return match[0].toUpperCase();
        });
    }
```
避免xss攻击的escapeHTML方法
```javascript
    function excapeHTML(target){
        return target.replace(/&/g, '&amp;').replace...
    }
```
pad方法：与trim方法相反，pad方法可以在字符串的某一端添加字符串，比如在日历之前补0
```javascript
    function pad(target, n){
        var zero = new Array(n).join('0');
        var str = zero + target;
        //从右边开始截取
        var result = substr(-n);
        return result;
    }
```

数组的扩展与修复
-
在IE6-8对ES5的数组方法主要通过修改数组原型来修复：
```javascript
    Array.prototype.indexOf = function(item, index){
        var n = this.length, i = --index;
        if(i < 0){
            i += n;
        }
        for(; i < n; i++){
            if(this[i] === item) return i;
        }
        return -1;
    }
```
实现indexOf可以获取数组相应元素之后，就可以实现其他很多方法.
flatten方法，用来拍平数组，返回一维数组
```javascript
    function flatten(target){
        var result = []
          len = target.length;
        for(var i = 0; i < len; i++){
            if($.isArray(arr[i])) {
                result.concat(flatten(arr[i]));
            } else {
                result.push(arr[i]);
            }
        } 
        return result;
    }
```
pluck方法: 取得数组上每个元素的指定属性，组成数组返回
```javascript
    function pluck(target, name){
        var result = [], 
           prop;
        target.forEach(function(item){
           prop = item[name];
           if(prop != null) result.push(prop);
        });
        return result;
    }
```
groupBy方法：根据指定条件进行分组
```javascript
    function groupBy(target, val){
        var result = {};
        var iterator = $.isFunction(val) ? val : function(obj){
            return obj[val];
        }
        target.forEach(function(value, index){
            var key = iterator(value);
            (result[key] || (result[key] = [])).push(value);
        });
        return result;
    }
```

函数的扩展与修复
-
bind方法：劫持作用域，并预先添加更多参数
```javascript
    Function.prototype.bind = function(context){
       if(arguments.length < 2 && context == void 0) return this;
       var _method = this, args = [].slice.call(arguments, 1);
       return function(){
           _method.apply(context, args.concat.apply(args, arguments));
       }
    }
```
使用bind，我们可以修改attacheEvent回调中的this问题，它总是指向window对象。
```javascript
  var addEvent = document.addEventListener ? 
       function(el, type, fn, capture){
           el.addEventListener(type, fn, capture)
       } :
       function(el, type, fn){
           el.attacheEvent('on'+ type, fn.bind(el, event))
       } :
```

类工厂
-
当我们想要一些私有属性，可以通过这样的方式
```javascript
    function A(){
        var count = 0;
        this.method = function(){
            return count;
        }
    }
```
实现继承最简单的方式就是拷贝继承，比如这个浅拷贝：
```javascript
  function extend(destination, source){
      for(var property in source)
      destination[property] = source[property];
      return destination;
  }
```
浅拷贝的缺点是无法通过`instanceof`验证。
第二种是原型继承。但是原型继承没有继承父类的类成员与特权成员。
如果需要继承类成员，可以通过这样：
```javascript
  function inherit(init, Parent, proto){
      function Son(){
          //先继承父类的特权成员
          Parent.apply(this, arguments);
          //再执行自己的构造器
          init.apply(this, arguments);  
      }
      Son.prototype = Object.create(Parent.prototype, {});
      extend(Son, Parent);
      return Son;
  }
```

es5属性描述符对OO库的冲击
-
Object提供了这些方法：

 * Object.keys
 * Object.getOwnPropertyNames
 * Object.getPrototypeOf
 * Object.defineProperty
 * ...
 
其中除了`Object.keys`之外，旧版本IE都无法模仿这些API。`Object.keys`用来遍历对象上非原型的属性。 

数据缓存系统
-
数据缓存系统主要有以下方式：

 * 属性标记法
 * 数组索引法
 * valueOf重写法
 * WeakMap关联法
 
jQuery1.2使用data和removeData方法，放在$.cache仓库上，为每个元素节点、文档对象分配一个UUID，UUID使用时间戳进行区分，挂载在windowData对象上。
```javascript
  var expando = 'jQuery' + (new Date().getTime()), uuid = 0, windowData = {};
  jQuery.extend({
      cache：{},
      data: function(elem, name, data){
          elem = elem == window ? windowData : elem;
          var id =  elem[expando];
          if(!id) id = elem[expando] = ++uuid;
          if(name && !jQuery.cache[id]) jQuery.cache[id] = {};
          if(data != undefined) jQuery.cache[id][name] = data;
          return name ? jQuery.cache[id][name] : id;
      }
  });
```
