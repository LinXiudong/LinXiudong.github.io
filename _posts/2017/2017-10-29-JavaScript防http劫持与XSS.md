---
layout: mypost
title: JavaScript防http劫持与XSS[转]
categories: [前端安全]
---

作为前端，一直以来都知道HTTP劫持与XSS跨站脚本（Cross-site scripting）、CSRF跨站请求伪造（Cross-site request forgery）。防御劫持最好的方法还是从后端入手，前端能做的实在太少。而且由于源码的暴露，攻击者很容易绕过我们的防御手段。但是这不代表我们去了解这块的相关知识是没意义的，本文的许多方法，用在其他方面也是大有作用。

## HTTP劫持、DNS劫持与XSS
先简单讲讲什么是 HTTP 劫持与 DNS 劫持。

**HTTP劫持**

什么是HTTP劫持呢，大多数情况是运营商HTTP劫持，当我们使用HTTP请求请求一个网站页面的时候，网络运营商会在正常的数据流中插入精心设计的网络数据报文，让客户端（通常是浏览器）展示“错误”的数据，通常是一些弹窗，宣传性广告或者直接显示某网站的内容，大家应该都有遇到过。

**DNS劫持**

DNS 劫持就是通过劫持了 DNS 服务器，通过某些手段取得某域名的解析记录控制权，进而修改此域名的解析结果，导致对该域名的访问由原IP地址转入到修改后的指定IP，其结果就是对特定的网址不能访问或访问的是假网址，从而实现窃取资料或者破坏原有正常服务的目的。

DNS 劫持比之 HTTP 劫持 更加过分，简单说就是我们请求的是 http://www.a.com/index.html ，直接被重定向了 http://www.b.com/index.html ，本文不会过多讨论这种情况。

**XSS跨站脚本**

XSS指的是攻击者利用漏洞，向 Web 页面中注入恶意代码，当用户浏览该页之时，注入的代码会被执行，从而达到攻击的特殊目的。

关于这些攻击如何生成，攻击者如何注入恶意代码到页面中本文不做讨论，只要知道如 HTTP 劫持 和 XSS 最终都是恶意代码在客户端，通常也就是用户浏览器端执行，本文将讨论的就是假设注入已经存在，如何利用 Javascript 进行行之有效的前端防护。


## 页面被嵌入 iframe 中，重定向 iframe
先来说说我们的页面被嵌入了 iframe 的情况。也就是，网络运营商为了尽可能地减少植入广告对原有网站页面的影响，通常会通过把原有网站页面放置到一个和原页面相同大小的 iframe 里面去，那么就可以通过这个 iframe 来隔离广告代码对原有页面的影响。

![iframe][1]

这种情况还比较好处理，我们只需要知道我们的页面是否被嵌套在 iframe 中，如果是，则重定向外层页面到我们的正常页面即可。

那么有没有方法知道我们的页面当前存在于 iframe 中呢？有的，就是 window.self 与 window.top 。

**window.self**
返回一个指向当前 window 对象的引用。

**window.top**
返回窗口体系中的最顶层窗口的引用。

对于非同源的域名，iframe 子页面无法通过 parent.location 或者 top.location 拿到具体的页面地址，但是可以写入 top.location ，也就是可以控制父页面的跳转。

两个属性分别可以又简写为 self 与 top，所以当发现我们的页面被嵌套在 iframe 时，可以重定向父级页面：

```js
if (self != top) {
    // 我们的正常页面
    var url = location.href;
    // 父级页面重定向
    top.location = url;
}
```

### 使用白名单放行正常 iframe 嵌套
当然很多时候，也许运营需要，我们的页面会被以各种方式推广，也有可能是正常业务需要被嵌套在 iframe 中，这个时候我们需要一个白名单或者黑名单，当我们的页面被嵌套在 iframe 中且父级页面域名存在白名单中，则不做重定向操作。

上面也说了，使用 top.location.href 是没办法拿到父级页面的 URL 的，这时候，需要使用document.referrer。

通过 document.referrer 可以拿到跨域 iframe 父页面的URL。

```js
// 建立白名单
var whiteList = [
    'www.aaa.com',
    'res.bbb.com'
];

if (self != top) {
    var
        // 使用 document.referrer 可以拿到跨域 iframe 父页面的 URL
        parentUrl = document.referrer,
        length = whiteList.length,
        i = 0;

    for(; i<length; i++){
        // 建立白名单正则
        var reg = new RegExp(whiteList[i],'i');

        // 存在白名单中，放行
        if(reg.test(parentUrl)){
            return;
        }
    }

    // 我们的正常页面
    var url = location.href;
    // 父级页面重定向
    top.location = url;
}
```

### 更改 URL 参数绕过运营商标记
这样就完了吗？没有，我们虽然重定向了父页面，但是在重定向的过程中，既然第一次可以嵌套，那么这一次重定向的过程中页面也许又被 iframe 嵌套了，真尼玛蛋疼。

当然运营商这种劫持通常也是有迹可循，最常规的手段是在页面 URL 中设置一个参数，例如 http://www.example.com/index.html?iframe_hijack_redirected=1 ，其中 iframe_hijack_redirected=1 表示页面已经被劫持过了，就不再嵌套 iframe 了。所以根据这个特性，我们可以改写我们的 URL ，使之看上去已经被劫持了：

```js
var flag = 'iframe_hijack_redirected';
// 当前页面存在于一个 iframe 中
// 此处需要建立一个白名单匹配规则，白名单默认放行
if (self != top) {
    var
        // 使用 document.referrer 可以拿到跨域 iframe 父页面的 URL
        parentUrl = document.referrer,
        length = whiteList.length,
        i = 0;

    for(; i<length; i++){
        // 建立白名单正则
        var reg = new RegExp(whiteList[i],'i');

        // 存在白名单中，放行
        if(reg.test(parentUrl)){
            return;
        }
    }

    var url = location.href;
    var parts = url.split('#');
    if (location.search) {
        parts[0] += '&' + flag + '=1';
    } else {
        parts[0] += '?' + flag + '=1';
    }
    try {
        console.log('页面被嵌入iframe中:', url);
        top.location.href = parts.join('#');
    } catch (e) {}
}
```
当然，如果这个参数一改，防嵌套的代码就失效了。所以我们还需要建立一个上报系统，当发现页面被嵌套时，发送一个拦截上报，即便重定向失败，也可以知道页面嵌入 iframe 中的 URL，根据分析这些 URL ，不断增强我们的防护手段，这个后文会提及。


## 内联事件及内联脚本拦截
在 XSS 中，其实可以注入脚本的方式非常的多，尤其是 HTML5 出来之后，一不留神，许多的新标签都可以用于注入可执行脚本。

列出一些比较常见的注入方式：
```html
<a href="javascript:alert(1)" ></a> (1)
<iframe src="javascript:alert(1)" />    (2)
<img src='x' onerror="alert(1)" />  (3)
<video src='x' onerror="alert(1)" ></video> (4)
<div onclick="alert(1)" onmouseover="alert(2)" ><div>   (5)
```
除去一些未列出来的非常少见生僻的注入方式，大部分都是 javascript:... 及内联事件 on*。

我们假设注入已经发生，那么有没有办法拦截这些内联事件与内联脚本的执行呢？

对于上面列出的 (1)(5) ，这种需要用户点击或者执行某种事件之后才执行的脚本，我们是有办法进行防御的。

### 浏览器事件模型
这里说能够拦截，涉及到了事件模型相关的原理。

我们都知道，标准浏览器事件模型存在三个阶段：

- 捕获阶段
- 目标阶段
- 冒泡阶段

对于一个这样 \<a href="javascript:alert(222)" \>\</a\> 的 a 标签而言，真正触发元素 alert(222) 是处于点击事件的目标阶段。
```js
<a href="javascript:alert(222)">Click Me</a>

<script>
    document.addEventListener('click', function(e) {
        alert(111);
    }, true);
</script>
```
点击上面的 click me ，先弹出 111 ，后弹出 222。

那么，我们只需要在点击事件模型的捕获阶段对标签内 javascript:... 的内容建立关键字黑名单，进行过滤审查，就可以做到我们想要的拦截效果。

对于 on* 类内联事件也是同理，只是对于这类事件太多，我们没办法手动枚举，可以利用代码自动枚举，完成对内联事件及内联脚本的拦截。

以拦截 a 标签内的 href="javascript:... 为例，我们可以这样写：
```js
// 建立关键词黑名单
var keywordBlackList = [
    'xss',
    'BAIDU_SSP__wrapper',
    'BAIDU_DSPUI_FLOWBAR'
];

document.addEventListener('click', function(e) {
    var code = "";

    // 扫描 <a href="javascript:"> 的脚本
    if (elem.tagName == 'A' && elem.protocol == 'javascript:') {
        var code = elem.href.substr(11);

        if (blackListMatch(keywordBlackList, code)) {
            // 注销代码
            elem.href = 'javascript:void(0)';
            console.log('拦截可疑事件:' + code);
        }
    }
}, true);

/**
 * [黑名单匹配]
 * @param  {[Array]} blackList [黑名单]
 * @param  {[String]} value    [需要验证的字符串]
 * @return {[Boolean]}         [false -- 验证不通过，true -- 验证通过]
 */
function blackListMatch(blackList, value) {
    var length = blackList.length,
        i = 0;

    for (; i < length; i++) {
        // 建立黑名单正则
        var reg = new RegExp(whiteList[i], 'i');

        // 存在黑名单中，拦截
        if (reg.test(value)) {
            return true;
        }
    }
    return false;
}
```

## 静态脚本拦截
XSS 跨站脚本的精髓不在于“跨站”，在于“脚本”。

通常而言，攻击者或者运营商会向页面中注入一个\<script\>脚本，具体操作都在脚本中实现，这种劫持方式只需要注入一次，有改动的话不需要每次都重新注入。

我们假定现在页面上被注入了一个 \<script src="http://attack.com/xss.js"\> 脚本，我们的目标就是拦截这个脚本的执行。

听起来很困难啊，什么意思呢。就是在脚本执行前发现这个可疑脚本，并且销毁它使之不能执行内部代码。

所以我们需要用到一些高级 API ，能够在页面加载时对生成的节点进行检测。

### MutationObserver
MutationObserver 是 HTML5 新增的 API，功能很强大，能够监测到页面 DOM 树的变换，并作出反应。给开发者们提供了一种能在某个范围内的 DOM 树发生变化时作出适当反应的能力。

MutationObserver() 该构造函数用来实例化一个新的Mutation观察者对象。
```js
MutationObserver(
  function callback
);
```
MutationObserver 在观测时并非发现一个新元素就立即回调，而是将一个时间片段里出现的所有元素，一起传过来。所以在回调中我们需要进行批量处理。而且，其中的 callback 会在指定的 DOM 节点(目标节点)发生变化时被调用。在调用时,观察者对象会传给该函数两个参数，第一个参数是个包含了若干个 MutationRecord 对象的数组，第二个参数则是这个观察者对象本身。

所以，使用 MutationObserver ，我们可以对页面加载的每个静态脚本文件，进行监控：
```js
// MutationObserver 的不同兼容性写法
var MutationObserver = window.MutationObserver || window.WebKitMutationObserver ||
window.MozMutationObserver;
// 该构造函数用来实例化一个新的 Mutation 观察者对象
// Mutation 观察者对象能监听在某个范围内的 DOM 树变化
var observer = new MutationObserver(function(mutations) {
    mutations.forEach(function(mutation) {
        // 返回被添加的节点,或者为null.
        var nodes = mutation.addedNodes;

        for (var i = 0; i < nodes.length; i++) {
            var node = nodes[i];
            if (/xss/i.test(node.src))) {
                try {
                    node.parentNode.removeChild(node);
                    console.log('拦截可疑静态脚本:', node.src);
                } catch (e) {}
            }
        }
    });
});

// 传入目标节点和观察选项
// 如果 target 为 document 或者 document.documentElement
// 则当前文档中所有的节点添加与删除操作都会被观察到
observer.observe(document, {
    subtree: true,
    childList: true
});
```
比如 \<script type="text/javascript" src="./xss/a.js"\>\</script\> 是页面加载一开始就存在的静态脚本（查看页面结构），我们使用 MutationObserver 可以在脚本加载之后，执行之前这个时间段对其内容做正则匹配，发现恶意代码则 removeChild() 掉，使之无法执行。

### 使用白名单对 src 进行匹配过滤
上面的代码中，我们判断一个js脚本是否是恶意的，用的是这一句：
```js
if (/xss/i.test(node.src)) {}
```
当然实际当中，注入恶意代码者不会那么傻，把名字改成 XSS 。所以，我们很有必要使用白名单进行过滤和建立一个拦截上报系统。
```js
// 建立白名单
var whiteList = [
    'www.aaa.com',
    'res.bbb.com'
];

/**
 * [白名单匹配]
 * @param  {[Array]} whileList [白名单]
 * @param  {[String]} value    [需要验证的字符串]
 * @return {[Boolean]}         [false -- 验证不通过，true -- 验证通过]
 */
function whileListMatch(whileList, value) {
    var length = whileList.length,
        i = 0;

    for (; i < length; i++) {
        // 建立白名单正则
        var reg = new RegExp(whiteList[i], 'i');

        // 存在白名单中，放行
        if (reg.test(value)) {
            return true;
        }
    }
    return false;
}

// 只放行白名单
if (!whileListMatch(blackList, node.src)) {
    node.parentNode.removeChild(node);
}
```
这里我们已经多次提到白名单匹配了，下文还会用到，所以可以这里把它简单封装成一个方法调用。


## 动态脚本拦截
上面使用 MutationObserver 拦截静态脚本，除了静态脚本，与之对应的就是动态生成的脚本。
```js
var script = document.createElement('script');
script.type = 'text/javascript';
script.src = 'http://www.example.com/xss/b.js';

document.getElementsByTagName('body')[0].appendChild(script);　
```
要拦截这类动态生成的脚本，且拦截时机要在它插入 DOM 树中，执行之前，**本来是**可以监听 Mutation Events 中的 DOMNodeInserted 事件的。

**Mutation Events 与 DOMNodeInserted**

**该特性已经从 Web 标准中删除，虽然一些浏览器目前仍然支持它，但也许会在未来的某个时间停止支持，请尽量不要使用该特性。**


### 重写 setAttribute 与 document.write
重写原生 Element.prototype.setAttribute 方法

在动态脚本插入执行前，监听 DOM 树的变化拦截它行不通，脚本仍然会执行。

那么我们需要向上寻找，在脚本插入 DOM 树前的捕获它，那就是创建脚本时这个时机。

假设现在有一个动态脚本是这样创建的：
```js
var script = document.createElement('script');
script.setAttribute('type', 'text/javascript');
script.setAttribute('src', 'http://www.example.com/xss/c.js');

document.getElementsByTagName('body')[0].appendChild(script);
```
而重写 Element.prototype.setAttribute 也是可行的：我们发现这里用到了 setAttribute 方法，如果我们能够改写这个原生方法，监听设置 src 属性时的值，通过黑名单或者白名单判断它，就可以判断该标签的合法性了。
```js
// 保存原有接口
var old_setAttribute = Element.prototype.setAttribute;

// 重写 setAttribute 接口
Element.prototype.setAttribute = function(name, value) {
    // 匹配到 <script src='xxx' > 类型
    if (this.tagName == 'SCRIPT' && /^src$/i.test(name)) {
        // 白名单匹配
        if (!whileListMatch(whiteList, value)) {
            console.log('拦截可疑模块:', value);
            return;
        }
    }

  // 调用原始接口
  old_setAttribute.apply(this, arguments);
};

// 建立白名单
var whiteList = [
    'www.yy.com',
    'res.cont.yy.com'
];

/**
 * [白名单匹配]
 * @param  {[Array]} whileList [白名单]
 * @param  {[String]} value    [需要验证的字符串]
 * @return {[Boolean]}         [false -- 验证不通过，true -- 验证通过]
 */
function whileListMatch(whileList, value) {
    var length = whileList.length,
        i = 0;

    for (; i < length; i++) {
        // 建立白名单正则
        var reg = new RegExp(whiteList[i], 'i');

        // 存在白名单中，放行
        if (reg.test(value)) {
            return true;
        }
    }
    return false;
}
```
重写 Element.prototype.setAttribute ，就是首先保存原有接口，然后当有元素调用 setAttribute 时，检查传入的 src 是否存在于白名单中，存在则放行，不存在则视为可疑元素，进行上报并不予以执行。最后对放行的元素执行原生的 setAttribute ，也就是 old_setAttribute.apply(this, arguments);。

上述的白名单匹配也可以换成黑名单匹配。

### 重写嵌套 iframe 内的 Element.prototype.setAttribute
当然，上面的写法如果 old_setAttribute = Element.prototype.setAttribute 暴露给攻击者的话，直接使用old_setAttribute 就可以绕过我们重写的方法了，所以这段代码必须包在一个闭包内。

当然这样也不保险，虽然当前窗口下的 Element.prototype.setAttribute 已经被重写了。但是还是有手段可以拿到原生的 Element.prototype.setAttribute ，只需要一个新的 iframe 。
```js
var newIframe = document.createElement('iframe');
document.body.appendChild(newIframe);

Element.prototype.setAttribute = newIframe.contentWindow.Element.prototype.setAttribute;
```
通过这个方法，可以重新拿到原生的 Element.prototype.setAttribute ，因为 iframe 内的环境和外层 window 是完全隔离的。

怎么办？我们看到创建 iframe 用到了 createElement，那么是否可以重写原生 createElement 呢？但是除了createElement 还有 createElementNS ，还有可能是页面上已经存在 iframe，所以不合适。

那就在每当新创建一个新 iframe 时，对 setAttribute 进行保护重写，这里又有用到 MutationObserver ：

```js
/**
 * 使用 MutationObserver 对生成的 iframe 页面进行监控，
 * 防止调用内部原生 setAttribute 及 document.write
 * @return {[type]} [description]
 */
function defenseIframe() {
    // 先保护当前页面
    installHook(window);
}

/**
 * 实现单个 window 窗口的 setAttribute保护
 * @param  {[BOM]} window [浏览器window对象]
 * @return {[type]}       [description]
 */
function installHook(window) {
    // 重写单个 window 窗口的 setAttribute 属性
    resetSetAttribute(window);

    // MutationObserver 的不同兼容性写法
    var MutationObserver = window.MutationObserver || window.WebKitMutationObserver || window.MozMutationObserver;

    // 该构造函数用来实例化一个新的 Mutation 观察者对象
    // Mutation 观察者对象能监听在某个范围内的 DOM 树变化
    var observer = new MutationObserver(function(mutations) {
        mutations.forEach(function(mutation) {
            // 返回被添加的节点,或者为null.
            var nodes = mutation.addedNodes;

            // 逐个遍历
            for (var i = 0; i < nodes.length; i++) {
                var node = nodes[i];

                // 给生成的 iframe 里环境也装上重写的钩子
                if (node.tagName == 'IFRAME') {
                    installHook(node.contentWindow);
                }
            }
        });
    });

    observer.observe(document, {
        subtree: true,
        childList: true
    });
}

/**
 * 重写单个 window 窗口的 setAttribute 属性
 * @param  {[BOM]} window [浏览器window对象]
 * @return {[type]} [description]
 */
function resetSetAttribute(window) {
    // 保存原有接口
    var old_setAttribute = window.Element.prototype.setAttribute;

    // 重写 setAttribute 接口
    window.Element.prototype.setAttribute = function(name, value) {
        //...
    };
}　
```
我们定义了一个 installHook 方法，参数是一个 window ，在这个方法里，我们将重写传入的 window 下的 setAttribute ，并且安装一个 MutationObserver ，并对此窗口下未来可能创建的 iframe 进行监听，如果未来在此 window 下创建了一个 iframe ，则对新的 iframe 也装上 installHook 方法，以此进行层层保护。


### 重写 document.write
根据上述的方法，我们可以继续挖掘一下，还有什么方法可以重写，以便对页面进行更好的保护。

document.write 是一个很不错选择，注入攻击者，通常会使用这个方法，往页面上注入一些弹窗广告。

我们可以重写 document.write，使用关键词黑名单对内容进行匹配。

什么比较适合当黑名单的关键字呢？我们可以看看一些广告很多的页面：

这里在页面最底部嵌入了一个 iframe ，里面装了广告代码，这里的最外层的 id 名id="BAIDU_SSP__wrapper_u2444091_0" 就很适合成为我们判断是否是恶意代码的一个标志，假设我们已经根据拦截上报收集到了一批黑名单列表：
```js
// 建立正则拦截关键词
var keywordBlackList = [
    'xss',
    'BAIDU_SSP__wrapper',
    'BAIDU_DSPUI_FLOWBAR'
];
```
接下来我们只需要利用这些关键字，对 document.write 传入的内容进行正则判断，就能确定是否要拦截document.write 这段代码。　

```javascript
// 建立关键词黑名单
var keywordBlackList = [
    'xss',
    'BAIDU_SSP__wrapper',
    'BAIDU_DSPUI_FLOWBAR'
];

/**
 * 重写单个 window 窗口的 document.write 属性
 * @param  {[BOM]} window [浏览器window对象]
 * @return {[type]}       [description]
 */
function resetDocumentWrite(window) {
    var old_write = window.document.write;

    window.document.write = function(string) {
        if (blackListMatch(keywordBlackList, string)) {
            console.log('拦截可疑模块:', string);
            return;
        }

        // 调用原始接口
        old_write.apply(document, arguments);
    }
}

/**
 * [黑名单匹配]
 * @param  {[Array]} blackList [黑名单]
 * @param  {[String]} value    [需要验证的字符串]
 * @return {[Boolean]}         [false -- 验证不通过，true -- 验证通过]
 */
function blackListMatch(blackList, value) {
    var length = blackList.length,
        i = 0;

    for (; i < length; i++) {
        // 建立黑名单正则
        var reg = new RegExp(whiteList[i], 'i');

        // 存在黑名单中，拦截
        if (reg.test(value)) {
            return true;
        }
    }
    return false;
}
```
我们可以把 resetDocumentWrite 放入上文的 installHook 方法中，就能对当前 window 及所有生成的 iframe 环境内的 document.write 进行重写了。


## 锁死 apply 和 call
接下来要介绍的这个是锁住原生的 Function.prototype.apply 和 Function.prototype.call 方法，锁住的意思就是使之无法被重写。

这里要用到 Object.defineProperty ，用于锁死 apply 和 call。

### Object.defineProperty
Object.defineProperty() 方法直接在一个对象上定义一个新属性，或者修改一个已经存在的属性， 并返回这个对象。
```js
Object.defineProperty(obj, prop, descriptor)
```
其中:

obj – 需要定义属性的对象

prop – 需被定义或修改的属性名

descriptor – 需被定义或修改的属性的描述符

我们可以使用如下的代码，让 call 和 apply 无法被重写。
```js
// 锁住 call
Object.defineProperty(Function.prototype, 'call', {
    value: Function.prototype.call,
    // 当且仅当仅当该属性的 writable 为 true 时，该属性才能被赋值运算符改变
    writable: false,
    // 当且仅当该属性的 configurable 为 true 时，该属性才能够被改变，也能够被删除
    configurable: false,
    enumerable: true
});
// 锁住 apply
Object.defineProperty(Function.prototype, 'apply', {
    value: Function.prototype.apply,
    writable: false,
    configurable: false,
    enumerable: true
});
```
为啥要这样写呢？其实还是与上文的 重写 setAttribute 有关。

虽然我们将原始 Element.prototype.setAttribute 保存在了一个闭包当中，但是还有奇技淫巧可以把它从闭包中给“偷出来”。

```js
(function() {})(
    // 保存原有接口
    var old_setAttribute = Element.prototype.setAttribute;
    // 重写 setAttribute 接口
    Element.prototype.setAttribute = function(name, value) {
        // 具体细节
        if (this.tagName == 'SCRIPT' && /^src$/i.test(name)) {}
        // 调用原始接口
        old_setAttribute.apply(this, arguments);
    };
)();
// 重写 apply
Function.prototype.apply = function(){
    console.log(this);
}
// 调用 setAttribute
document.getElementsByTagName('body')[0].setAttribute('data-test','123');　
```
猜猜上面一段会输出什么？
```js
function setAttribute() { [native code] }
```
居然返回了原生 setAttribute 方法！

这是因为我们在重写 Element.prototype.setAttribute 时最后有 old_setAttribute.apply(this, arguments);这一句，使用到了 apply 方法，所以我们再重写 apply ，输出 this ，当调用被重写后的 setAttribute 就可以从中反向拿到原生的被保存起来的 old_setAttribute 了。

这样我们上面所做的嵌套 iframe 重写 setAttribute 就毫无意义了。

使用上面的 Object.defineProperty 可以锁死 apply 和 类似用法的 call 。使之无法被重写，那么也就无法从闭包中将我们的原生接口偷出来。这个时候才算真正意义上的成功重写了我们想重写的属性。


## 建立拦截上报
防御的手段也有一些了，接下来我们要建立一个上报系统，替换上文中的 console.log() 日志。

上报系统有什么用呢？因为我们用到了白名单，关键字黑名单，这些数据都需要不断的丰富，靠的就是上报系统，将每次拦截的信息传到服务器，不仅可以让我们程序员第一时间得知攻击的发生，更可以让我们不断收集这类相关信息以便更好的应对。

这里的示例我用 nodejs 搭一个十分简易的服务器接受 http 上报请求。

先定义一个上报函数：
```js
/**
* 自定义上报 -- 替换页面中的 console.log()
* @param  {[String]} name  [拦截类型]
* @param  {[String]} value [拦截值]
*/
function hijackReport(name, value) {
    var img = document.createElement('img'),
        hijackName = name,
        hijackValue = value.toString(),
        curDate = new Date().getTime();

    // 上报
    img.src = 'http://www.reportServer.com/report/?msg=' + hijackName + '&value=' + hijackValue + '&time=' + curDate;
}
```
假定我们的服务器地址是 www.reportServer.com 这里，我们运用 img.src 发送一个 http 请求到服务器http://www.reportServer.com/report/ ，每次会带上我们自定义的拦截类型，拦截内容以及上报时间。

用 Express 搭 nodejs 服务器并写一个简单的接收路由：

```js
var express = require('express');
var app = express();

app.get('/report/', function(req, res) {
    var queryMsg = req.query.msg,
        queryValue = req.query.value,
        queryTime = new Date(parseInt(req.query.time));

    if (queryMsg) {
        console.log('拦截类型：' + queryMsg);
    }

    if (queryValue) {
        console.log('拦截值：' + queryValue);
    }

    if (queryTime) {
        console.log('拦截时间：' + req.query.time);
    }
});

app.listen(3002, function() {
    console.log('HttpHijack Server listening on port 3002!');
});
```
运行服务器，当有上报发生，我们将会接收到如下数据：
![node][2]
好接下来就是数据入库，分析，添加黑名单，使用 nodejs 当然拦截发生时发送邮件通知程序员等等，这些就不再做展开。

## HTTPS 与 CSP
最后再简单谈谈 HTTPS 与 CSP。其实防御劫持最好的方法还是从后端入手，前端能做的实在太少。而且由于源码的暴露，攻击者很容易绕过我们的防御手段。

**CSP**

CSP 即是 Content Security Policy，翻译为内容安全策略。这个规范与内容安全有关，主要是用来定义页面可以加载哪些资源，减少 XSS 的发生。

**HTTPS**

能够实施 HTTP 劫持的根本原因，是 HTTP 协议没有办法对通信对方的身份进行校验以及对数据完整性进行校验。如果能解决这个问题，则劫持将无法轻易发生。

HTTPS，是 HTTP over SSL 的意思。SSL 协议是 Netscape 在 1995 年首次提出的用于解决传输层安全问题的网络协议，其核心是基于公钥密码学理论实现了对服务器身份认证、数据的私密性保护以及对数据完整性的校验等功能。

因为与本文主要内容关联性不大，关于更多 CSP 和 HTTPS 的内容可以自行谷歌。

[文章来源](http://www.cnblogs.com/coco1s/p/5777260.html)



[1]: 01.png
[2]: 02.png
