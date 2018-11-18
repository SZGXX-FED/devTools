# web 前端性能优化解决方案

## 一、前言

web 产品从简单的静态展示为主的网站，变为功能越来越复杂应用程序，整个 web 程序的体积变得越来越大。为了提升 web 产品的用户体验，减少用户等待时间，有必要在软件开发的过程中，考虑采取相应的前端优化策略。

另外，产品在交付验收前，通常需要做并发测试。为了达到并发测试的要求，除了提高服务器硬件配置、优化服务端代码外，也需要我们充分考虑前端代码的优化。

本文主要从`网络层面`和`渲染层面`两个大的维度来论述 web 应用的性能优化的解决方案。

## 二、网络层面的优化

网络层面，主要是对 HTTP 请求的优化，包括以下两个方向：

- 1.减少请求次数
- 2.减少单次请求所花费的时间

### 1. 使用构建工具压缩合并静态资源

在日常开发中，我们通过使用诸如 Grunt, gulp, webpack, rollup, FIS3 等构建工具来对静态资源文件（JavaScript, CSS, 图片等）进行 `压缩` 与 `合并`。

引入构建工具后，压缩文件，能够减少静态资源的体积，从而减少单次 HTTP 请求所花的时间。合并资源文件，能够减少 HTTP 请求的次数。

### 2. 选择合适的图片格式

目前比较常用的 Web 图片格式有 JPG、PNG、WebP、Base64、SVG 等。根据使用场景，合理选择图片格式，达到降低图片体积的目的。

#### 2.1 JPEG/JPG

JPG 最大的特点是`有损压缩`。这种高效的压缩算法使它成为了一种非常轻巧的图片格式。另一方面，即使被称为“有损”压缩，JPG 的压缩方式仍然是一种高质量的压缩方式：当我们把图片体积压缩至原有体积的 50% 以下时，JPG 仍然可以保持住 60% 的品质。此外，JPG 格式以 24 位存储单个图，可以呈现多达 1600 万种颜色，足以应对大多数场景下对色彩的要求。

缺点： 不支持`透明度`。

适用场景：JPG 适用于呈现色彩丰富的图片，经常作为`大的背景图`、`轮播图`或 `Banner 图`出现。

#### 2.2 PNG-8 与 PNG-24

PNG（可移植网络图形格式）是一种`无损压缩的`高保真的图片格式。8 和 24，这里都是二进制数的位数，8 位的 PNG 最多支持 256 种颜色，而 24 位的可以呈现约 1600 万种颜色。

PNG 图片具有比 JPG 更强的色彩表现力，对线条的处理更加细腻，对`透明度`有良好的支持。

理论上来说，当你追求最佳的显示效果、并且不在意文件体积大小时，是推荐使用 PNG-24 的。

但实践当中，为了规避体积的问题，我们一般不用 PNG 去处理较复杂的图像。当我们遇到适合 PNG 的场景时，也会优先选择更为小巧的 `PNG-8`。

缺点：唯一的 BUG 就是`体积太大`。

适用场景：`小的 Logo`、`颜色简单且对比强烈`的图片或背景等。

#### 2.3 SVG

SVG（可缩放矢量图形）是一种`基于 XML 语法`的图像格式。它和本文提及的其它图片种类有着本质的不同：SVG 对图像的处理不是基于像素点，而是是`基于对图像的形状描述`。

SVG 与 PNG 和 JPG 相比，文件`体积更小`，`可压缩性更强`。

作为矢量图，它最显著的优势还是在于`图片可无限放大而不失真`这一点上。这使得 SVG 即使是被放到视网膜屏幕上，也可以一如既往地展现出较好的成像品质——1 张 SVG 足以适配 n 种分辨率。

SVG 是文本文件，因此既可以像写代码一样定义 SVG，把它写在 HTML 里、成为 DOM 的一部分，也可以把对图形的描述写入以 .svg 为后缀的独立文件（SVG 文件在使用上与普通图片文件无异）。这使得 SVG 文件可以被非常多的工具读取和修改，具有较强的灵活性。

缺点：渲染成本比较高，这点对性能来说是很不利的。

#### 2.4 雪碧图: 最经典的小图标解决方案

雪碧图、CSS Sprites 是指将`小图标`和`背景图像`合并到一张图片上，然后利用 CSS 的`背景定位`来显示其中的每一部分的技术。

雪碧图，被运用于众多使用`大量小图标`的网页应用之上。它可取图像的一部分来使用，使得使用一个图像文件替代多个小文件成为可能。相较于一个小图标一个图像文件，单独一张图片所需的 HTTP 请求更少，对内存和带宽更加友好。

适用场景：有大量的小图标

#### 2.5 Base64

Base64 并非一种图片格式，而是一种编码方式。Base64 和雪碧图一样，是作为`小图标`解决方案而存在的。

和雪碧图一样，Base64 图片的出现，也是为了减少加载网页图片时对服务器的`请求次数`，从而提升网页性能。Base64 是作为雪碧图的补充而存在的。

Base64 是一种用于传输 8Bit 字节码的编码方式，通过对图片进行 Base64 编码，以直接将编码结果写入 HTML 或者写入 CSS，从而减少 HTTP 请求的次数。

Base64 编码后，图片大小会膨胀为原文件的 4/3（这是由 Base64 的编码原理决定的）。如果我们把大图也编码到 HTML 或 CSS 文件中，后者的体积会明显增加，即便我们减少了 HTTP 请求，也无法弥补这庞大的体积带来的性能开销，得不偿失。
在传输非常小的图片的时候，Base64 带来的文件体积膨胀、以及浏览器解析 Base64 的时间开销，与它节省掉的 HTTP 请求开销相比，可以忽略不计，这时候才能真正体现出它在性能方面的优势。

适用场景：

- 图片的实际尺寸很小（一般不超过 2kb ）
- 图片无法以雪碧图的形式与其它小图结合（合成雪碧图仍是主要的减少 HTTP 请求的途径，Base64 是雪碧图的补充）
- 图片的更新频率非常低（不需我们重复编码和修改文件内容，维护成本较低）

### 3.服务端开启 Gizp

常用的 web 服务器，比如 Nignx, Apache，IIS，都可以通过简单地更改配置文件来开启 Gzip 压缩功能。

以 Nignx 为例：

```conf
gzip on;
gzip_min_length 1k;
zip_types text/plain application/x-javascript text/css application/xml text/javascript image/jpeg image/gif image/png;
```

### 4.合理利用浏览器缓存（服务端代码）

缓存可以减少网络 IO 消耗，提高访问速度。浏览器缓存是一种操作简单、效果显著的前端性能优化手段。

浏览器缓存机制有四个方面，它们按照获取资源时请求的优先级依次排列如下：

- Memory Cache
- Service Worker Cache
- HTTP Cache
- Push Cache

#### 4.1 HTTP 缓存

HTTP 缓存是我们日常开发中最为熟悉的一种缓存机制。它又分为`强缓存`和`协商缓存`。优先级较高的是强缓存，在命中强缓存失败的情况下，才会走协商缓存。

##### 4.1.1 强缓存

`强缓存`是利用 http 响应头中的 `Expires` 和 `Cache-Control` 两个字段来控制的。强缓存中，当请求再次发出时，浏览器会根据其中的 `expires` 和 `cache-control` 判断目标资源是否“命中”强缓存，若命中则直接从缓存中获取资源，不会再与服务端发生通信。

命中强缓存的情况下，返回的 HTTP 状态码为 `200`。

```
expires: Fri, 16 Nov 2018 10:55:00 GMT
```

`expires` 是一个时间戳，接下来如果我们试图再次向服务器请求资源，浏览器就会先对比`本地时间`和 `expires 的时间戳`，如果本地时间小于 expires 设定的过期时间，那么就直接去缓存中取这个资源。

expires 最大的问题在于对“本地时间”的依赖。如果服务端和客户端的时间设置可能不同，或者我直接手动去把客户端的时间改掉，那么 expires 将无法达到我们的预期。

考虑到 expires 的局限性，HTTP1.1 新增了 `Cache-Control` 字段来完成 expires 的任务。`Cache-Control` 可以视作是 expires 的完全替代方案。在当下的前端实践里，我们继续使用 expires 的唯一目的就是向下兼容。

```
cache-control: max-age=31536000
```

`Cache-Control` 相对于 `expires` 更加准确，它的优先级也更高。当 `Cache-Control` 与 `expires` 同时出现时，我们以 `Cache-Control` 为准。

##### 4.1.2 协商缓存：浏览器与服务器合作之下的缓存策略

`协商缓存`依赖于`服务端`与`浏览器`之间的通信。

协商缓存机制下，浏览器需要向服务器去询问缓存的相关信息，进而判断是重新发起请求、下载完整的响应，还是从本地获取缓存的资源。

如果服务端提示缓存资源未改动（Not Modified），资源会被重定向到浏览器缓存，这种情况下网络请求对应的状态码是 304.

###### Last-Modified

`协商缓存`的实现是通过控制 `Last-Modified` 和 `Etag` 两个字段来实现的。

`Last-Modified` 是一个`时间戳`，如果我们启用了协商缓存，它会在首次请求时随着 Response Headers 返回：

```
last-modified: Fri, 26 Jan 2018 06:58:11 GMT
```

随后我们每次请求时，会带上一个叫 `If-Modified-Since` 的时间戳字段，它的值正是上一次 response 返回给它的 last-modified 值：

```
If-Modified-Since: Fri, 26 Jan 2018 06:58:11 GMT
```

服务器接收到这个时间戳后，会比对该时间戳和资源在服务器上的最后修改时间是否一致，从而判断资源是否发生了变化。如果发生了变化，就会返回一个完整的响应内容，并在 Response Headers 中添加新的 Last-Modified 值；否则，返回如上图的 304 响应，Response Headers 不会再添加 Last-Modified 字段。

使用 Last-Modified 存在一些弊端，这其中最常见的就是这样两个场景：

我们编辑了文件，但文件的内容没有改变。服务端并不清楚我们是否真正改变了文件，它仍然通过最后编辑时间进行判断。因此这个资源在再次被请求时，会被当做新资源，进而引发一次完整的响应——不该重新请求的时候，也会重新请求。

当我们修改文件的速度过快时（比如花了 100ms 完成了改动），由于 If-Modified-Since 只能检查到以秒为最小计量单位的时间差，所以它是感知不到这个改动的——该重新请求的时候，反而没有重新请求了。

这两个场景其实指向了同一个 bug——服务器并没有正确感知文件的变化。为了解决这样的问题，`Etag` 作为 Last-Modified 的补充出现了。

###### Etag

Etag 是由`服务器`为每个资源生成的`唯一的标识字符串`，这个标识字符串是`基于文件内容编码`的，只要文件内容不同，它们对应的 Etag 就是不同的，反之亦然。因此 Etag 能够精准地感知文件的变化。

Etag 和 Last-Modified 类似，当首次请求时，我们会在响应头里获取到一个最初的标识符字符串。下一次请求时，请求头里就会带上一个值相同的、名为 `If-None-Match` 的字符串供服务端比对了。

Etag 的生成过程需要服务器额外付出开销，会影响服务端的性能，这是它的弊端。因此启用 Etag 需要我们审时度势。正如我们刚刚所提到的——Etag 并不能替代 Last-Modified，它只能作为 Last-Modified 的补充和强化存在。 Etag 在感知文件变化上比 Last-Modified 更加准确，优先级也更高。当 Etag 和 Last-Modified 同时存在时，以 Etag 为准。

### 5.合理使用本地存储：从 Cookie 到 Web Storage、IndexDB

#### 5.1 Cookie

`Cookie` 的本职工作并非本地存储，而是`“维持状态”`。

在 Web 开发的早期，人们亟需解决的一个问题就是`状态管理`的问题：HTTP 协议是一个`无状态协议`，服务器接收客户端的请求，返回一个响应，服务器并没有记录下关于客户端的任何信息。

Cookie 说白了就是一个存储在浏览器里的一个小小的文本文件，它附着在 HTTP 请求上，在浏览器和服务器之间“飞来飞去”。它可以携带用户信息，当服务器检查 Cookie 的时候，便可以获取到客户端的状态。

##### Cookie 的缺点

1）Cookie 最大只能有 4KB。当 Cookie 超过 4KB 时，它将面临被裁切的命运。

2）过量的 Cookie 会带来巨大的性能浪费。

Cookie 是紧跟域名的。我们通过响应头里的 `Set-Cookie` 指定要存储的 Cookie 值。默认情况下，domain 被设置为设置 Cookie 页面的主机名，我们也可以手动设置 domain 的值：

```
Set-Cookie: name=xiuyan; domain=xiuyan.me
```

`同一个域名下的所有请求，都会携带 Cookie`。即使是请求一张图片或者一个 CSS 文件，也会携带 Cookie。Cookie 虽然小，请求却可以有很多，随着请求的叠加，这样的不必要的 Cookie 带来的开销将是无法想象的。

随着前端应用复杂度的提高，Cookie 也渐渐演化为了一个“存储多面手”——它不仅仅被用于维持状态，还被塞入了一些乱七八糟的其它信息，被迫承担起了本地存储的“重任”。在没有更好的本地存储解决方案的年代里，Cookie 小小的身体里承载了 4KB 内存所不能承受的压力。

#### 5.2 Web Storage

Web Storage 是 HTML5 专门为浏览器存储而提供的数据存储机制。它又分为 `Local Storage` 与 `Session Storage`。

##### Web Storage 的特性

1）存储容量大，存储容量可以达到 5-10M 之间。
2）仅位于浏览器端，不与服务端发生通信。

##### Local Storage 与 Session Storage 的区别

两者的区别在于生命周期与作用域的不同。

- 生命周期：Local Storage 是持久化的本地存储，存储在其中的数据是永远不会过期的，使其消失的唯一办法是手动删除；而 Session Storage 是临时性的本地存储，它是会话级别的存储，当会话结束（页面被关闭）时，存储内容也随之被释放。
- 作用域：Local Storage、Session Storage 和 Cookie 都遵循同源策略。但 Session Storage 特别的一点在于，即便是相同域名下的两个页面，只要它们不在同一个浏览器窗口中打开，那么它们的 Session Storage 内容便无法共享。

##### 自带的增删改查方法

Web Storage 保存的数据内容和 Cookie 一样，是文本内容，以`键值对`的形式存在，值必须是`字符串`类型(如果是对象，需要先用`JSON.stringify()`转换)。与 cookie 相比，Web Storage 有自带的增删改查接口方便使用。

```js
// 存储数据：setItem()
localStorage.setItem("user_name", "xiuyan");
// 读取数据： getItem()
localStorage.getItem("user_name");
// 删除某一键名对应的数据： removeItem()
localStorage.removeItem("user_name");
// 清空数据记录：clear()
localStorage.clear();

// 对象需要转成字符串，才能存储
var userInfo = {
  userName: "Jack",
  gender: "male",
  age: 12
};
localStorage.setItem("user_info", JSON.stringify(userInfo));
```

##### 应用场景

1）Local Storage：在存储方面没有什么特别的限制，理论上 Cookie 无法胜任的、可以用简单的键值对来存取的数据存储任务，都可以交给 Local Storage 来做。可以用来存储一些内容稳定的资源，比如图片内容丰富的电商网站会用它来存储 Base64 格式的图片字符串以及一些不经常更新的 CSS、JS 等静态资源。

2）Session Storage 更适合用来存储生命周期和它同步的会话级别的信息。这些信息只适用于当前会话，当你开启新的会话时，它也需要相应的更新或释放。比如微博的 Session Storage 就主要是存储你本次会话的浏览足迹。

Web Storage 是对 Cookie 的拓展，它只能用于存储少量的简单数据。当遇到大规模的、结构复杂的数据时，需要用到 IndexDB。

#### 5.3 IndexDB

IndexDB 是一个运行在浏览器上的`非关系型数据库`。

理论上来说，IndexDB 是没有存储上限的（一般来说不会小于 250M）。它不仅可以存储`字符串`，还可以存储`二进制数据`。

1.打开/创建一个 IndexDB 数据库（当该数据库不存在时，open 方法会直接创建一个名为 xiaoceDB 新数据库）。

```js
// 后面的回调中，我们可以通过 event.target.result 拿到数据库实例
let db;
// 参数 1 位数据库名，参数 2 为版本号
const request = window.indexedDB.open("xiaoceDB", 1);
// 使用 IndexDB 失败时的监听函数
request.onerror = function(event) {
  console.log("无法使用 IndexDB");
};
// 成功
request.onsuccess = function(event) {
  // 此处就可以获取到 db 实例
  db = event.target.result;
  console.log("你打开了 IndexDB");
};
```

2.创建一个 object store（object store 对标到数据库中的“表”单位）。

```js
// onupgradeneeded 事件会在初始化数据库/版本发生更新时被调用，我们在它的监听函数中创建 object store
request.onupgradeneeded = function(event) {
  let objectStore;
  // 如果同名表未被创建过，则新建 test 表
  if (!db.objectStoreNames.contains("test")) {
    objectStore = db.createObjectStore("test", { keyPath: "id" });
  }
};
```

3.构建一个事务来执行一些数据库操作，像增加或提取数据等。

```js
// 创建事务，指定表格名称和读写权限
const transaction = db.transaction(["test"], "readwrite");
// 拿到 Object Store 对象
const objectStore = transaction.objectStore("test");
// 向表格写入数据
objectStore.add({ id: 1, name: "xiuyan" });
```

4.通过监听正确类型的事件以等待操作完成。

```js
// 操作成功时的监听函数
transaction.oncomplete = function(event) {
  console.log("操作成功");
};
// 操作失败时的监听函数
transaction.onerror = function(event) {
  console.log("这里有一个Error");
};
```

###### 应用场景：

在 IndexDB 中，我们可以创建多个数据库，一个数据库中创建多张表，一张表中存储多条数据——这足以 hold 住复杂的结构性数据。IndexDB 可以看做是 LocalStorage 的一个升级，当数据的复杂度和规模上升到了 LocalStorage 无法解决的程度，可以考虑使用 IndexDB。

## 三、渲染层面的优化

### 1. 前置知识补充：浏览器渲染过程解析

在浏览器里，每一个页面的首次渲染都经历了如下阶段（图中箭头不代表串行，有一些操作是并行进行的，下文会说明）：

```
解析 HTML（DOM tree） => 计算样式 (CSSOM tree) => 计算图层布局 (Layout of the render tree)=> 绘制图层(Painting the render tree) => 整合图层，得到页面
```

#### 1.1 解析 HTML

在这一步浏览器执行了所有的加载解析逻辑，渲染引擎开始解析 HTML 文档，转换树中的标签到 DOM 节点，它被称为 `DOM 树`。在解析 HTML 的过程中发出了页面渲染所需的各种`外部资源请求`。

#### 1.2 计算样式

浏览器将识别并加载所有的 CSS 样式信息，解析 CSS（包括外部 CSS 文件和样式元素）创建 `CSSOM 树`。CSSOM 的解析过程与 DOM 的解析过程是并行的。CSSOM 与 DOM 结合，之后我们得到的就是`渲染树`（Render tree ）（:after :before 这样的伪元素会在这个环节被构建到 DOM 树中）。

#### 1.3 计算图层布局

页面中所有元素的相对位置信息，大小等信息均在这一步得到计算。

从根节点递归调用，计算每一个元素的大小、位置等，给每个节点所应该出现在屏幕上的精确坐标，我们便得到了基于渲染树的`布局渲染树`（Layout of the render tree）

#### 1.4 绘制图层

在这一步中浏览器会根据我们的 DOM 代码结果，把每一个页面`图层`转换为`像素`，并对所有的`媒体文件`进行`解码`。

遍历渲染树，每个节点将使用 UI 后端层来绘制。整个过程叫做绘制渲染树（Painting the render tree）

#### 1.5 整合图层，得到页面

最后一步浏览器会合并合各个图层，将数据由 CPU 输出给 GPU 最终绘制在屏幕上。

### 2. 基于渲染流程的 CSS 优化解决方案

看如下规则,：

```css
#myList li {
}
```

习惯了从左到右阅读的文字阅读方式，会本能地以为浏览器也是从左到右匹配 CSS 选择器的，因此会推测这个选择器并不会费多少力气：#myList 是一个 id 选择器，它对应的元素只有一个，查找起来应该很快。定位到了 myList 元素，等于是缩小了范围后再去查找它后代中的 li 元素。

但是，CSS 引擎查找样式表，对每条规则都按 `从右到左` 的顺序去匹配。浏览器必须遍历页面上每个 li 元素，并且每次都要去确认这个 li 元素的父元素 id 是不是 myList。

有不少人直接用 `通配符` 清除默认样式:

```css
* {
}
```

它会匹配所有元素，所以浏览器必须去遍历每一个元素。

CSS 选择器书写习惯，可以为我们带来非常可观的性能提升。根据上面的分析，我们至少可以总结出如下性能提升的方案：

- 避免使用通配符，只对需要用到的元素进行选择。

- 关注可以通过继承实现的属性，避免重复匹配重复定义。

- 少用`标签选择器`。如果可以，用`类选择器`替代。

- 不要画蛇添足，id 和 class 选择器不应该被多余的标签选择器拖后腿。

```css
/* 高效的写法 */
#title {
}

/* 画蛇添足的写法 */
div#title {
}
```

- 减少嵌套。`后代选择器`的开销是最高的，因此我们应该尽量将选择器的深度降到最低（最高不要超过 3 层），尽可能使用`类`来关联每一个标签元素。

### 3. 优化 CSS 与 JS 的加载顺序，告别阻塞

#### 3.1 CSS 放在 `<head>` 内

当我们开始解析 HTML 后、解析到 link 标签或者 style 标签时，CSS 才登场，CSSOM 的构建才开始。很多时候，DOM 不得不等待 CSSOM。我们需要将它尽早、尽快地下载到客户端，以便缩短首次渲染的时间。因此，需要将 CSS 放在 head 标签里。

#### 3.2 异步加载 JavaScript 文件

JS 引擎是独立于渲染引擎存在的。我们的 JS 代码在文档的何处插入，就在何处执行。当 HTML 解析器遇到一个 script 标签时，它会暂停渲染过程，将控制权交给 JS 引擎。JS 引擎对内联的 JS 代码会直接执行，对外部 JS 文件还要先获取到脚本、再进行执行。等 JS 引擎运行完毕，浏览器又会把控制权还给渲染引擎，继续 CSSOM 和 DOM 的构建。 因此与其说是 JS 把 CSS 和 HTML 阻塞了，不如说是 JS 引擎抢走了渲染引擎的控制权。

给 script 标签，添加 defer 属性，告诉浏览器在等待脚本可用期间不阻止其它的工作。JS 会被异步加载的，执行会被推迟。等整个文档解析完成、DOMContentLoaded 事件即将被触发时，被标记了 defer 的 JS 文件才会开始依次执行。

```html
<script defer src="lib.js"></script>
<script defer src="index.js"></script>
```

### 4.DOM 优化

#### 4.1 DOM 优化原理

```
把 DOM 和 JavaScript 各自想象成一个岛屿，它们之间用收费桥梁连接。——《高性能 JavaScript》
```

JS 引擎和渲染引擎（浏览器内核）是独立实现的。当我们用 JS 去操作 DOM 时，本质上是 JS 引擎和渲染引擎之间进行了“跨界交流”。这个“跨界交流”的实现并不简单，它依赖了桥接接口作为“桥梁”。每操作一次 DOM（不管是为了修改还是仅仅为了访问其值），都要过一次“桥”。过“桥”的次数一多，就会产生比较明显的性能问题，因此，需要尽可能地 `减少 DOM 操作`。

对 DOM 的修改会引发它外观（样式）上的改变时，就会触发 `回流(reflow)` 或 `重绘(repaint)`。这个过程本质上还是因为我们对 DOM 的修改触发了渲染树（Render Tree）的变化所导致的。

- 回流：当我们对 DOM 的修改引发了 DOM 几何尺寸的变化（比如修改元素的宽、高或隐藏元素等）时，浏览器需要重新计算元素的几何属性（其他元素的几何属性和位置也会因此受到影响），然后再将计算的结果绘制出来。这个过程就是回流（也叫重排）。

- 重绘：当我们对 DOM 的修改导致了样式的变化、却并未影响其几何属性（比如修改了颜色或背景色）时，浏览器不需重新计算元素的几何属性、直接为该元素绘制新的样式（跳过了上图所示的回流环节）。这个过程叫做重绘。

#### 4.2 使用 DOM Fragment: 减少 DOM 操作

DocumentFragment 接口表示一个没有父级文件的最小文档对象。它被当做一个轻量版的 Document 使用，用于存储已排好版的或尚未打理好格式的 XML 片段。因为 DocumentFragment 不是真实 DOM 树的一部分，它的变化不会引起 DOM 树的重新渲染的操作（reflow），且不会导致性能等问题。

示例：往 id 为 container 的元素里插入一大段 HTML 内容。

```js
// 普通方式
for (var count = 0; count < 10000; count++) {
  document.getElementById("container").innerHTML +=
    "<span>我是一个小测试</span>";
}
```

```js
// 使用 DOM Fragment,减少 DOM 操作。
let container = document.getElementById("container");
// 创建一个DOM Fragment对象作为容器
let content = document.createDocumentFragment();
for (let count = 0; count < 10000; count++) {
  // span此时可以通过DOM API去创建
  let oSpan = document.createElement("span");
  oSpan.innerHTML = "我是一个小测试";
  // 像操作真实DOM一样操作DOM Fragment对象
  content.appendChild(oSpan);
}
// 内容处理好了,最后再触发真实DOM的更改
container.appendChild(content);
```

#### 4.3 缓存 DOM 元素的几何属性：避免频繁改动

当一个 DOM 元素的几何属性(比如 width、height、padding、margin、left、top、border )发生变化时，所有和它相关的节点（比如父子节点、兄弟节点等）的几何属性都需要进行重新计算，它会带来巨大的计算量。

另外，获取一些特定属性的值，比如 offsetTop、offsetLeft、 offsetWidth、offsetHeight、scrollTop、scrollLeft、scrollWidth、scrollHeight、clientTop、clientLeft、clientWidth、clientHeight，
因为浏览器需要通过 `即时计算` 得到这些属性，也会进行回流。

当我们调用了 `getComputedStyle` 方法，或者 IE 里的 `currentStyle` 时，也会触发`回流`。原理是一样的，都为求一个“即时性”和“准确性”。

需求：通过多次计算得到一个元素的布局位置。

A.普通的写法：

```js
// 获取el元素
const el = document.getElementById("el");
// 这里循环判定比较简单，实际中或许会拓展出比较复杂的判定需求
for (let i = 0; i < 10; i++) {
  el.style.top = el.offsetTop + 10 + "px";
  el.style.left = el.offsetLeft + 10 + "px";
}
```

每次循环都需要获取多次“敏感属性”，是比较糟糕的。我们可以将其以 JS 变量的形式`缓存`起来，待计算完毕再提交给浏览器发出重计算请求：

B.更高效的写法：

```js
// 缓存offsetLeft与offsetTop的值
const el = document.getElementById("el");
let offLeft = el.offsetLeft,
  offTop = el.offsetTop;

// 在JS层面进行计算
for (let i = 0; i < 10; i++) {
  offLeft += 10;
  offTop += 10;
}

// 一次性将计算结果应用到DOM上
el.style.left = offLeft + "px";
el.style.top = offTop + "px";
```

#### 4.4 使用`类名`去合并样式：避免逐条改变样式

A. 普通写法：前者每次单独操作，都去触发一次渲染树更改，从而导致相应的回流与重绘过程。

```js
const container = document.getElementById("container");
container.style.width = "100px";
container.style.height = "200px";
container.style.border = "10px solid red";
container.style.color = "red";
```

B. 更高效的写法：合并之后将所有的更改一次性发出。

```css
.basic_style {
  width: 100px;
  height: 200px;
  border: 10px solid red;
  color: red;
}
```

```js
const container = document.getElementById("container");
container.classList.add("basic_style");
```

#### 4.5 将 DOM 离线化

我们上文所说的回流和重绘，都是在“该元素位于页面上”的前提下会发生的。一旦我们给元素设置 `display: none`，将其从页面上“拿掉”，那么我们的后续操作，将无法触发回流与重绘——这个将元素“拿掉”的操作，就叫做 `DOM 离线化`。

A. 普通写法

```js
const container = document.getElementById("container");
container.style.width = "100px";
container.style.height = "200px";
container.style.border = "10px solid red";
container.style.color = "red";
```

B. 更高效的写法：将 DOM 离线

```js
let container = document.getElementById("container");
// DOM 离线化
container.style.display = "none";
container.style.width = "100px";
container.style.height = "200px";
container.style.border = "10px solid red";
container.style.color = "red";

container.style.display = "block";
```

虽然，拿掉一个元素再把它放回去，也会触发一次昂贵的 `回流(reflow)`。但我们把它拿下来了，后续不管操作这个元素多少次，每一步的操作成本都会非常低。当我们只需要进行很少的 DOM 操作时，DOM 离线化的优越性确实不太明显。一旦操作频繁起来，这“拿掉”和“放回”的开销都将会是非常值得的。

### 5. 事件的节流（throttle）与防抖（debounce）

scroll 事件、resize 事件、鼠标事件（比如 mousemove、mouseover 等）、键盘事件（keyup、keydown 等）都存在被频繁触发的风险。

频繁触发回调导致的大量计算会引发页面的抖动甚至卡顿。为了规避这种情况，我们需要一些手段来控制事件被触发的频率。就是在这样的背景下，throttle（事件节流）和 debounce（事件防抖）出现了。

#### 5.1 事件节流 throttle

在某段时间内，不管你触发了多少次回调，都只认第一次，并在计时结束时给予响应。

```js
// fn是我们需要包装的事件回调, interval是时间间隔的阈值
function throttle(fn, interval) {
  // last为上一次触发回调的时间
  let last = 0;

  // 将throttle处理结果当作函数返回
  return function() {
    // 保留调用时的this上下文
    let context = this;
    // 保留调用时传入的参数
    let args = arguments;
    // 记录本次触发回调的时间
    let now = +new Date();

    // 判断上次触发的时间和本次触发的时间差是否小于时间间隔的阈值
    if (now - last >= interval) {
      // 如果时间间隔大于我们设定的时间间隔阈值，则执行回调
      last = now;
      fn.apply(context, args);
    }
  };
}

// 用throttle来包装scroll的回调
document.addEventListener(
  "scroll",
  throttle(() => console.log("触发了滚动事件"), 1000)
);
```

#### 5.2 事件防抖 debounce

在某段时间内，不管你触发了多少次回调，都只认最后一次。

```js
// fn是我们需要包装的事件回调, delay是每次推迟执行的等待时间
function debounce(fn, delay) {
  // 定时器
  let timer = null;

  // 将debounce处理结果当作函数返回
  return function() {
    // 保留调用时的this上下文
    let context = this;
    // 保留调用时传入的参数
    let args = arguments;

    // 每次事件被触发时，都去清除之前的旧定时器
    if (timer) {
      clearTimeout(timer);
    }
    // 设立新定时器
    timer = setTimeout(function() {
      fn.apply(context, args);
    }, delay);
  };
}

// 用debounce来包装scroll的回调
document.addEventListener(
  "scroll",
  debounce(() => console.log("触发了滚动事件"), 1000)
);
```

#### 5.3 用 Throttle 来优化 Debounce

debounce 的问题在于它“太有耐心了”。如果用户的操作十分频繁——他每次都不等 debounce 设置的 delay 时间结束就进行下一次操作，于是每次 debounce 都为该用户重新生成定时器，回调函数被延迟了不计其数次。频繁的延迟会导致用户迟迟得不到响应，用户同样会产生“这个页面卡死了”的观感。

为了避免弄巧成拙，我们需要借力 throttle 的思想，打造一个“有底线”的 debounce——等你可以，但我有我的原则：delay 时间内，我可以为你重新生成定时器；但只要 delay 的时间到了，我必须要给用户一个响应。这个 throttle 与 debounce “合体”思路，已经被很多成熟的前端库应用到了它们的加强版 throttle 函数的实现中：

```js
// fn是我们需要包装的事件回调, delay是时间间隔的阈值
function throttle(fn, delay) {
  // last为上一次触发回调的时间, timer是定时器
  let last = 0,
    timer = null;
  // 将throttle处理结果当作函数返回

  return function() {
    // 保留调用时的this上下文
    let context = this;
    // 保留调用时传入的参数
    let args = arguments;
    // 记录本次触发回调的时间
    let now = +new Date();

    // 判断上次触发的时间和本次触发的时间差是否小于时间间隔的阈值
    if (now - last < delay) {
      // 如果时间间隔小于我们设定的时间间隔阈值，则为本次触发操作设立一个新的定时器
      clearTimeout(timer);
      timer = setTimeout(function() {
        last = now;
        fn.apply(context, args);
      }, delay);
    } else {
      // 如果时间间隔超出了我们设定的时间间隔阈值，那就不等了，无论如何要反馈给用户一次响应
      last = now;
      fn.apply(context, args);
    }
  };
}

// 用新的throttle包装scroll的回调
document.addEventListener(
  "scroll",
  throttle(() => console.log("触发了滚动事件"), 1000)
);
```
