# H5 History API简单记录

[TOC]

H5里新增了很多API，关于历史记录History的API可以实现较多的骚操作。比如页面前进、后腿、无刷新更改地址栏链接、无刷新跳转等。

> 浏览器历史记录可以看作一个后进先出的「栈」，用户每点开一个新网页都会在栈顶加一条记录，叫「入栈」，每次点击「后退」按钮都会移除最上面的那条记录，叫做「出栈」。浏览器显示的自然是最顶端的内容。

## 事件

### popstate 事件

当用户点击浏览器的「前进」、「后退」按钮时，就会触发`popstate`事件。你可以监听这一事件，从而作出反应。

```javascript
window.addEventListener("popstate", function(e) {
    var state = e.state;
    // do something...
  
  	// e.state.id == 2;  //请看pushState的例子
});
```

这里`e.state`就是下面`pushState`时传入的第一个参数，是某条历史记录的state对象一个拷贝。。

## API列表

### pushState

假设当前页面为`https://github.com`，执行下面语句会让地址栏变成`https://github.com/stelalae`，但页面不会刷新，甚至不会检测目标页面是否存在，即不会在新地址页面发起任何请求。

```javascript
window.history.pushState(null, null, "/stelalae");

window.history.pushState(null, null, "?id=1"); // 支持query string的形式参数
```

`pushState`函数之后，会往浏览器的历史记录中添加一条新记录，同时改变地址栏的地址内容。它可以接收三个参数，分别为：

1. 一个对象或者字符串，用于描述新记录的一些特性。这个参数会被一并添加到历史记录中，以供以后使用。这个参数是开发者根据自己的需要自由给出的。
2. 一个字符串，代表新页面的标题。当前基本上所有浏览器都会忽略这个参数。
3. 一个字符串，代表新页面的相对地址。

```
var state = {
    id: 2,
};
window.history.pushState(state, "My Profile", "/stelalae");
```

在某种意义上， `pushState()` 与  `window.location = "#foo"` 等设置hash方式类似，二者都会在当前页面创建并激活新的历史记录。但 `pushState()` 具有如下几条优点：

- 新的 URL 可以是与当前URL同源的任意URL ，而设置 window.location 仅当你只修改了哈希值时才保持同一个文件。即改变query string后是否会导致刷新。
- 如果需要，可以不必改变URL就能创建一条历史记录。而设置 `window.location = "#foo"`；只有在当前哈希不是 `#foo` 的情况下， 才会创建一个新的历史记录项。即是否会改变window.history.length的值。
- 我们可以为新的历史记录项关联任意数据。而基于哈希值的方式，则必须将所有相关数据编码到一个短字符串里。即pushState可以通过state去设置任意数据，而window.location必须将数据编码放在query string后。

### replaceState

如果不想添加历史记录，即替换当前的记录，比如loading页面、登录页面等，则使用`replaceState`，其参数和`pushState`一样。

## 其他说明

### URL 的限制

为了安全考虑，新 URL 必须和当前 URL 在同一个域名下。例如，你不能把地址改成 Google 的首页。否则不怀好心的人就可以把地址改成网银等关键网站的地址，来迷惑用户了。



## 应用举例

### 全站 AJAX，并使浏览器能够抓取 AJAX 页面

这个可以干啥用？一个比较常用的场景就是，配合 AJAX。

假设一个页面左侧是若干导航链接，右侧是内容，同时导航时只有右侧的内容需要更新，那么刷新整个页面无疑是浪费的。这时我们可以使用 AJAX 来拉取右面的数据。但是如果仅仅这样，地址栏是不会改变的，用户无法前进、后退，也无法收藏当前页面或者把当前页面分享给他人；搜索引擎抓取也有困难。这时，就可以使用 HTML5 的 History API 来解决这个问题。

思路：首先绑定click事件。当用户点击一个链接时，通过preventDefault函数防止默认的行为（页面跳转），同时读取链接的地址（如果有 jQuery，可以写成$(this).attr('href')），把这个地址通过pushState塞入浏览器历史记录中，再利用 AJAX 技术拉取（如果有 jQuery，可以使用$.get方法）这个地址中真正的内容，同时替换当前网页的内容。

为了处理用户前进、后退，我们监听popstate事件。当用户点击前进或后退按钮时，浏览器地址自动被转换成相应的地址，同时popstate事件发生。在事件处理函数中，我们根据当前的地址抓取相应的内容，然后利用 AJAX 拉取这个地址的真正内容，呈现，即可。

最后，整个过程是不会改变页面标题的，可以通过直接对document.title赋值来更改页面标题。





参考链接：

1. [HTML5 简介（三）：利用 History API 无刷新更改地址栏](https://www.renfei.org/blog/html5-introduction-3-history-api.html)
2. [简单聊聊H5的pushState与replaceState](https://juejin.im/post/5a7332b25188257a6d634f8d)

