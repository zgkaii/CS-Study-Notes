> https://developer.mozilla.org/zh-CN/docs/Web/HTML

## 剖析HTML文档

学习了一些HTML元素的基础知识，这些元素单独一个是没有意义的。现在我们来学习这些特定元素是怎么被结合起来，从而形成一个完整的HTML页面的：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>我的测试站点</title>
  </head>
  <body>
    <p>这是我的页面</p>
  </body>
</html>
```

分析如下:

1. ```
   <!DOCTYPE html>
   ```

   : 声明文档类型. 很久以前，早期的HTML(大约1991年2月)，文档类型声明类似于链接，规定了HTML页面必须遵从的良好规则，能自动检测错误和其他有用的东西。使用如下：

   ```html
   <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
   "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
   ```

   然而这种写法已经过时了，这些内容已成为历史。只需要知道 `<!DOCTYPE html>`是最短有效的文档声明。

2. `<html></html>`: `<html>`元素。这个元素包裹了整个完整的页面，是一个根元素。

3. `<head></head>`: `<head>元素`. 这个元素是一个容器，它包含了所有你想包含在HTML页面中但不想在HTML页面中显示的内容。这些内容包括你想在搜索结果中出现的关键字和页面描述，CSS样式，字符集声明等等。以后的章节能学到更多关于<head>元素的内容。

4. `<meta charset="utf-8">`: 这个元素设置文档使用utf-8字符集编码，utf-8字符集包含了人类大部分的文字。基本上他能识别你放上去的所有文本内容。毫无疑问要使用它，并且它能在以后避免很多其他问题。

5. `<title></title>`: 设置页面标题，出现在浏览器标签上，当你标记/收藏页面时它可用来描述页面。

6. `<body></body>`: `<body>`元素。 包含了你访问页面时所有显示在页面上的内容，文本，图片，音频，游戏等等。

## 剖析一个 HTML 元素

让我们进一步探讨我们的段落元素：

![img](https://mdn.mozillademos.org/files/16475/element.png)

这个元素的主要部分有：

1. **开始标签**（Opening tag）：包含元素的名称（本例为 p），被左、右角括号所包围。表示元素从这里开始或者开始起作用 —— 在本例中即段落由此开始。
2. **结束标签**（Closing tag）：与开始标签相似，只是其在元素名之前包含了一个斜杠。这表示着元素的结尾 —— 在本例中即段落在此结束。初学者常常会犯忘记包含结束标签的错误，这可能会产生一些奇怪的结果。
3. **内容**（Content）：元素的内容，本例中就是所输入的文本本身。
4. **元素**（Element）：开始标签、结束标签与内容相结合，便是一个完整的元素。

## 属性

元素也可以拥有属性，如下：

![&amp;amp;amp;lt;p class="editor-note">我的猫咪脾气爆&amp;amp;amp;lt;/p>](https://mdn.mozillademos.org/files/16476/attribute.png)

属性包含元素的额外信息，这些信息不会出现在实际的内容中。在上述例子中，这个class属性给元素赋了一个识别的名字（id），这个名字此后可以被用来识别此元素的样式信息和其他信息。

一个属性必须包含如下内容：

1. 一个空格，在属性和元素名称之间。(如果已经有一个或多个属性，就与前一个属性之间有一个空格。)
2. 属性名称，后面跟着一个等于号。
3. 一个属性值，由一对引号“ ”引起来。

元素`<a>`是锚，它使被标签包裹的内容成为一个超链接。此元素也可以添加大量的属性，其中几个如下：

- `href`: 这个属性声明超链接的web地址，当这个链接被点击浏览器会跳转至href声明的web地址。例如：`href="https://www.mozilla.org/"`。
- `title`: 标题`title`属性为超链接声明额外的信息，比如你将链接至的那个页面。例如：`title="The Mozilla homepage"`。当鼠标悬停在超链接上面时，这部分信息将以工具提示的形式显示。
- `target`: 目标`target`属性用于指定链接如何呈现出来。例如，`target="_blank"`将在新标签页中显示链接。如果你希望在当前标签页显示链接，忽略这个属性即可。

## 在HTML中应用CSS和JavaScript

如今，几乎你使用的所有网站都会使用 [CSS](https://developer.mozilla.org/zh-CN/docs/Glossary/CSS) 让网页更加炫酷，使用[JavaScript](https://developer.mozilla.org/zh-CN/docs/Glossary/JavaScript)让网页有交互功能，比如视频播放器，地图，游戏以及更多功能。这些应用在网页中很常见，它们分别使用 `<link>`元素以及`<script>`元素。

-   `<link>`元素经常位于文档的头部。这个link元素有2个属性，rel="stylesheet"表明这是文档的样式表，而 href包含了样式表文件的路径：

```html
<link rel="stylesheet" href="my-css-file.css">
```


* `<script>` 部分没必要非要放在文档头部；实际上，把它放在文档的尾部（在 </body>标签之前）是一个更好的选择，这样可以确保在加载脚本之前浏览器已经解析了HTML内容（如果脚本加载某个不存在的元素，浏览器会报错）。

```html
<script src="my-js-file.js"></script>
```

**注意**： `<script>`元素看起来像一个空元素，但它并不是，因此需要一个结束标记。您还可以选择将脚本放入`<script>`元素中，而不是指向外部脚本文件。

## 统一资源定位符(URL)与路径(path)快速入门

要全面地了解链接目标，你需要了解统一资源定位符和文件路径。本小节将介绍相关的信息。

统一资源定位符（英文：**U**niform **R**esource **L**ocator，简写：URL）是一个定义了在网络上的位置的一个文本字符串。例如 Mozilla 的中文主页定位在 `https://www.mozilla.org/zh-CN/`.

URL使用路径查找文件。路径指定文件系统中您感兴趣的文件所在的位置。看一下一个简单的目录结构的例子（源码可查看 [创建超链接（creating-hyperlinks](https://github.com/roy-tian/learning-area/tree/master/html/introduction-to-html/creating-hyperlinks)）。）![A simple directory structure. The parent directory is called creating-hyperlinks and contains two files called index.html and contacts.html, and two directories called projects and pdfs, which contain an index.html and a project-brief.pdf file, respectively](https://mdn.mozillademos.org/files/12409/simple-directory.png)

此目录结构的**根目录**称为`creating-hyperlinks`。当在网站上工作时， 你会有一个包含整个网站的目录。在根目录，我们有一个`index.html`和一个`contacts.html`文件。在真实的网站上，`index.html` 将会成为我们的主页或登录页面。

我们的根目录还有两个目录—— `pdfs` 和`projects`，它们分别包含一个 PDF (`project-brief.pdf`) 文件和一个`index.html` 文件。请注意你可以有两个`index.html`文件，前提是他们在不同的目录下，许多网站就是如此。第二个`index.html`或许是项目相关信息的主登录界面。

- **指向当前目录：**如果`index.html`（目录顶层的`index.html`）想要包含一个超链接指向`contacts.html`，你只需要指定想要链接的文件名，因为它与当前文件是在同一个目录的. 所以你应该使用的URL是`contacts.html`:

  ```html
  <p>要联系某位工作人员，请访问我们的 <a href="contacts.html">联系人页面</a>。</p>
  ```

- **指向子目录：**如果`index.html` （目录顶层`index.html`）想要包含一个超链接指向 `projects/index.html`，您需要先进入`projects/`项目目录，然后指明要链接到的文件`index.html`。 通过指定目录的名称，然后是正斜杠，然后是文件的名称。因此您要使用的URL是`projects/index.html`：

  ```html
  <p>请访问 <a href="projects/index.html">项目页面</a>。</p>
  ```

- **指向上级目录：** 如果你想在`projects/index.html`中包含一个指向`pdfs/project-brief.pdf`的超链接，你必须先返回上级目录，然后再回到`pdf`目录。“返回上一个目录级”使用两个英文点号表示 — `..` — 所以你应该使用的URL是 `../pdfs/project-brief.pdf`：

  ```html
  <p>点击打开 <a href="../pdfs/project-brief.pdf">项目简介</a>。</p>
  ```

注意：如果需要的话，你可以将这些功能的多个例子和复杂的url结合起来。例如：`../../../complex/path/to/my/file.html`.

### 绝对URL和相对URL

**绝对URL**：指向由其在Web上的绝对位置定义的位置，包括 [protocol](https://developer.mozilla.org/zh-CN/docs/Glossary/Protocol)（协议） 和 [domain name](https://developer.mozilla.org/zh-CN/docs/Glossary/域名)（域名）。像下面的例子，如果`index.html`页面上传到`projects`这一个目录。并且`projects`目录位于web服务站点的根目录，web站点的域名为`http://www.example.com`，那么这个页面就可以通过`http://www.example.com/projects/index.html`访问（或者通过`http://www.example.com/projects/`来访问，因为在没有指定特定的URL的情况下，大多数web服务会默认访问加载`index.html`这类页面）

不管绝对URL在哪里使用，它总是指向确定的相同位置。

**相对URL**：指向与您链接的文件相关的位置，更像我们在前面一节中所看到的位置。例如，如果我们想从示例文件链接`http://www.example.com/projects/index.html`转到相同目录下的一个PDF文件，URL就是文件名URL——例如`project-brief.pdf`——没有其他的信息要求。如果PDF文件能够在`projects`的子目录`pdfs`中访问到，相对路径就是`pdfs/project-brief.pdf`（对应的绝对URL是`http://www.example.com/projects/pdfs/project-brief.pdf`）

一个相对URL将指向不同的位置，这取决于它所在的文件所在的位置——例如，如果我们把`index.html`文件从`projects`目录移动到Web站点的根目录（最高级别，而不是任何目录中），里面的`pdfs/project-brief.pdf`相对URL将会指向`http://www.example.com/pdfs/project-brief.pdf`，而不是`http://www.example.com/projects/pdfs/project-brief.pdf`

> 尽可能使用相对链接。