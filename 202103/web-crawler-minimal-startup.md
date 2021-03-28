# 网络爬虫最小化启动

# Web Crawler: Minimal Startup

## 2021 年 3 月 27 日

## Mar 27 2021

---

### 前言

这是一篇关于爬虫的教程，但是并不从爬虫讲起，而是讲爬虫需要的前置知识：HTML和HTTP。从计算机网络体系结构的角度讲，HTML和HTTP都属于应用层（除此，也有更精确的划分）的规范或协议，即计算机网络的最上层。

当我们说一种东西属于“协议”，即在说这是人为规定的，目的是达成一种共识，使得大家交流方便。所以，细节处不必问“为什么”，如要追溯，多半有不少历史遗留的原因，只是那时候这样规定了，所以后来人就按这个规定办事。

我们假设读者具有[Markdown](https://daringfireball.net/projects/markdown/)标记的基本知识，举个例子（注意，以下的md代码可以复制到类似[Typora](https://typora.io/)这样的md编辑器下打开，以查看视觉效果）。

```
# 这是一级标题
## 这是二级标题
###### 最小可以有六级标题

随便写一个段落。

> 这是一段引用。

**我要加粗这段字体**，*而这一段是斜体*。

超链接是这样写，请导航到：[百度](https://baidu.com)

引用网络上的一张图片：![IG](https://wx2.sinaimg.cn/mw1024/002i6acdgy1gowje93nqgj61900u07wk02.jpg)
```

相应的，HTML也可以看作一门标记语言，上面的Markdown标记换做HTML标记，可以写成如下格式，另外，最好把`<DOCTYPE html>`写在第一行，让浏览器知道这是一个HTML文件。

```html
<DOCTYPE html>
<h1>这是一级标题</h1>
<h2>这是二级标题</h2>
<h6>最小可以有六级标题</h6>
<p>
  随便写一个段落
</p>
<blockquote>
  这是一段引用。
</blockquote>
<p>
  <b>我要加粗这段字体</b>，<i>而这一段是斜体</i>。
</p>
<p>
  超链接是这样写的，请导航到：<a href="https://baidu.com">百度</a>
</p>
<p>
  引用网络上的一张图片：<img src="https://wx2.sinaimg.cn/mw1024/002i6acdgy1gowje93nqgj61900u07wk02.jpg">
</p>
```

把上面的内容复制到文本编辑器，修改后缀为`.html`，用任何浏览器打开它，你都将看到一个HTML页面。事实上，你用浏览器打开的任何网站几乎都是基于这样的架构。（网页之所以能呈现丰富的内容，是因为还用到了其它语言来做美工和交互，但其本质还是HTML标记）。

### HTML：更多标记

HTML和Markdown最大的区别之一是，HTML可以为某个标签加上唯一的`id`标志或者标记它属于某一类的class标志，比如：

```html
<DOCTYPE html>
<h1 id="whatever">这个标题有个唯一标志</h1>
<p class="hhh">
  这个段落有一个属性，名字叫“hhh”。
</p>
```

给标签加上`id`或`class`的目的是通过标记它们来为它们添加独特的样式或互动效果，比如，你希望某一些一级标题居中，而另一些一级标题靠左，你就需要给它们赋值不同的属性，然后通过样式语言CSS来调整。

**在爬虫的时候，我们就需要通过根据标签+class或id来获取其中的内容，这是爬虫的关键。不论你用什么语言，什么工具，本质就是去找到想要的标签、class或id名称，然后通过这个名称爬取下来。**

比如，来看著名的Python爬虫库BeautifulSoup写的一段代码：

```python
# web变量是已经获取的网页文本
web.find('div', attrs={'class': 'popular_tags'}).find_all('a')
```

这就是说，在`web`这个页面，我们需要爬去第一个属性名为`popular_tags`，标签名为`div`的标签下，所有的`a`标签。

这里之所以有什么多点`.`，是因为HTML标记通常是嵌套的，一个标签里套着许多其它标签，我们多数时候只想要爬取内层的一些标签内的值。

> 思考题：
>
> 1. 为什么爬虫的时候，根据id爬取返回的是单个值，而根据标签或class爬取通常会返回一串列表？
> 2. 如果有一个class下有多个标签，你根据class名称爬取到的内容，什么时候是一串列表，什么时候是单个文本？

### HTTP：网络传输协议

如果说HTML是静态的文本标记协议，那么HTTP就是动态的文本传输协议，这意味着HTTP的参与方总是有两个：客户端（用户）和服务端（服务器）。客户端发送请求，服务端处理请求并给出相应。

HTTP相对HTML来说更难理解，为了能够更好地、深入浅出地写好这一部分，笔者[特地去浏览了一遍《HTTP/2基础教程》](https://cescdf.com/blog/202103/reading-http2-basic)，以确保内容的正确性。

最早期的HTTP协议几乎只需要实现一个功能：让客户端用户能够从服务器获取（get）到HTML文档，这个请求类型现在被归类名为get，这也是目前实际上最流行的HTTP请求。

### get

get请求由用户发出，由服务器响应。举个例子，如果你在浏览器输入栏输入`www.baidu.com`，浏览器实际上是向这个域名所代表的服务器发出了请求，然后这个请求结果返回到你的本地电脑上，由浏览器负责把结果呈现为人类可读到页面。这整个过程非常复杂，涉及到域名的查找，连接的建立，样式的解析，这些所有复杂的过程都由浏览器包办了。这整个过程非常快，以至于常常让人意识不到其中的复杂性。

对于爬虫，我们仅需要知道要先发送这样类似的请求，才能获取到你想要的文本就可以了。在Python里，这些请求方法由`requests`封装好，例如，你只需要调用如下语句，就能把相应url的HTML文本（text）存到变量r里，当然还包括其他信息，比如[返回的状态码](https://cescdf.com/blog/202103/http-state-code-summary)（status_code），编码（encoding）的方式，请求的头部信息（headers）等等。

```python
>>> r = requests.get('https://api.github.com/user', auth=('user', 'pass'))
>>> r.status_code
200
>>> r.headers['content-type']
'application/json; charset=utf8'
>>> r.encoding
'utf-8'
>>> r.text
'{"type":"User"...'
>>> r.json()
{'private_gists': 419, 'total_private_repos': 77, ...}
```

你有时候也需要向你的请求添加头部，以模拟自身是一个浏览器，这样一些网站才不会屏蔽你的请求，比如：

```python
>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1;Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36'}

>>> r = requests.get(url, headers=headers)
```

一个常见的Python爬虫流程就是用requests库获得文本，然后用BeautifulSoup库解析文本（提取标签tag、class、id等），这就是最简单的爬虫。

### post

另一个最常用的HTTP发送请求的方法就是post，post相比于get的唯一区别就是，post可以在发送请求时携带比较大量的数据，最典型的场景是携带你的账号密码发送，这样服务器才可以验证你的信息，从而允许登陆。我们有时候需要通过post请求登陆到某个页面（比如京东、淘宝），这样它才能允许你查看商品信息页面，从而爬取相应的数据。

你必须知道post发送的目标地址。这需要你首先手动登陆，看一看地址是多少。一个常用的技巧是，打开浏览器（比如Chrome）的开发者工具（通常在右上角，设置的子菜单里），找到Network选项，打开它。然而，在你点击“登陆”的瞬间，看看浏览器发送了哪个post的请求，你可以在请求的信息中找到这样的url。

下面是requests提供的关于post 的一个demo：

```python
>>> payload = {'key1': 'value1', 'key2': 'value2'}

>>> r = requests.post("https://httpbin.org/post", data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```

payload就是你需要携带的信息，必须以键值对的形式出现，比如` {'account': 'dfnjkebj', 'password': '123456789'}`。

### 小结

以上就是网页爬虫需要的几乎全部前置知识了，掌握了这些，再去读[requests库的文档](https://requests.readthedocs.io/en/master/)，就会相对轻松一些。爬虫的知识是无穷无尽的，各种网页会设置各种各样的反爬措施，要成为高级爬虫工程师，就需要在这样的攻防中不断历练，才能掌握无数的爬虫和反爬技巧。