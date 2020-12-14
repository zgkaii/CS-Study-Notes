> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript

## JavaScript 在页面上做了什么？

- [HTML](https://developer.mozilla.org/zh-CN/docs/Glossary/HTML)是一种标记语言，用来结构化我们的网页内容并赋予内容含义，例如定义段落、标题和数据表，或在页面中嵌入图片和视频。
- [CSS](https://developer.mozilla.org/zh-CN/docs/Glossary/CSS) 是一种样式规则语言，可将样式应用于 HTML 内容， 例如设置背景颜色和字体，在多个列中布局内容。
- [JavaScript](https://developer.mozilla.org/zh-CN/docs/Glossary/JavaScript) 是一种脚本语言，可以用来创建动态更新的内容，控制多媒体，制作图像动画，还有很多。（好吧，虽然它不是万能的，但可以通过简短的代码来实现神奇的功能。）

浏览器在读取一个网页时，代码（HTML, CSS 和 JavaScript）将在一个运行环境（浏览器标签页）中得到执行。就像一间工厂，将原材料（代码）加工为一件产品（网页）。

![img](https://mdn.mozillademos.org/files/13504/execution.png)

在 HTML 和 CSS 集合组装成一个网页后，浏览器的 JavaScript 引擎将执行 JavaScript 代码。这保证了当 JavaScript 开始运行之前，网页的结构和样式已经就位。

这样很好，因为JavaScript 最普遍的用处是通过 DOM API（见上文）动态修改 HTML 和 CSS 来更新用户界面 （user interface）。如果 JavaScript 在 HTML 和 CSS 就位之前加载运行，就会引发错误。

### 解释代码 vs 编译代码

作为程序员，你或许听说过这两个术语：**解释（interpret)**和 **编译(compile)**。在解释型语言中，**代码自上而下运行**，且实时返回运行结果。代码在由浏览器执行前，不需要将其转化为其他形式。代码将直接以文本格式（text form）被接收和处理。

相对的，编译型语言需要先将代码转化（编译）成另一种形式才能运行。比如 C/C++ 先被编译成汇编语言，然后才能由计算机运行。程序将以二进制的格式运行，这些二进制内容是由程序源代码产生的。

**JavaScript 是轻量级解释型语言**。浏览器接受到JavaScript代码，并以代码自身的文本格式运行它。技术上，几乎所有 JavaScript 转换器都运用了一种叫做即时编译（just-in-time compiling）的技术；当 JavaScript 源代码被执行时，它会被编译成二进制的格式，使代码运行速度更快。尽管如此，JavaScript 仍然是一门解释型语言，因为编译过程发生在代码运行中，而非之前。

### 动态代码 vs 静态代码

“**动态**”一词既适用于客户端 JavaScript，又适用于描述服务器端语言。是指通过按需生成新内容来更新 web 页面 / 应用，使得不同环境下显示不同内容。服务器端代码会在服务器上动态生成新内容，例如从数据库中提取信息。而客户端 JavaScript 则在用户端浏览器中动态生成新内容，比如说创建一个新的 HTML 表格，用从服务器请求到的数据填充，然后在网页中向用户展示这个表格。两种情况的意义略有不同，但又有所关联，且两者（服务器端和客户端）经常协同作战。

没有动态更新内容的网页叫做“**静态**”页面**，**所显示的内容不会改变。

### 内部 JavaScript



1. 首先，下载示例文件 [apply-javascript.html](https://github.com/roy-tian/learning-area/blob/master/javascript/introduction-to-js-1/what-is-js/apply-javascript.html)。放在一个好记的文件夹里。

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="utf-8">
    <title>使用 JavaScript 的示例</title>
  </head>
  <body>
    <button>点我呀</button>
    <script>

    </script>
  </body>
</html>
```

2. 分别在浏览器和文本编辑器中打开这个文件。你会看到这个 HTML 文件创建了一个简单的网页，其中有一个可点击按钮。

3. 然后转到文本编辑器，在`</body>`标签结束前插入以下代码：

```html
<script>

  // 在此编写 JavaScript 代码

</script>
```

4. 下面，在`<script>`元素中添加一些 JavaScript 代码，这个页面就能做一些更有趣的事。在“/ /在此编写 JavaScript 代码”一行下方添加以下代码：

```js
document.addEventListener("DOMContentLoaded", function() {
  function createParagraph() {
    let para = document.createElement('p');
    para.textContent = '你点击了这个按钮！';
    document.body.appendChild(para);
  }

  const buttons = document.querySelectorAll('button');

  for(let i = 0; i < buttons.length ; i++) {
    buttons[i].addEventListener('click', createParagraph);
  }
});
```

5. 保存文件并刷新浏览器，然后你会发现，点击按钮文档下方将会添加一个新段落。

### 外部 JavaScript

这很不错，但是能不能把 JavaScript 代码放置在一个外部文件中呢？现在我们来研究一下。

1. 首先，在刚才的 HTML 文件所在的目录下创建一个名为 `script.js` 的新文件。请确保扩展名为 `.js`，只有这样才能被识别为 JavaScript 代码。

2. 将`<script>`元素替换为：

   ```html
   <script src="script.js" async></script>
   ```

3. 在`script.js`文件中，添加下面的脚本：

   ```js
   function createParagraph() {
     let para = document.createElement('p');
     para.textContent = '你点击了这个按钮！';
     document.body.appendChild(para);
   }
   
   const buttons = document.querySelectorAll('button');
   
   for(let i = 0; i < buttons.length ; i++) {
     buttons[i].addEventListener('click', createParagraph);
   } 
   ```

4. 保存并刷新浏览器，你会发现二者完全一样。但是现在我们把 JavaScript 写进了一个外部文件。这样做一般会使代码更加有序，更易于复用，且没有了脚本的混合，HTML 也会更加易读，因此这是个好的习惯。

### 内联 JavaScript 处理器

注意，有时候你会遇到在 HTML 中存在着一丝真实的 JavaScript 代码。它或许看上去像这样：

```js
function createParagraph() {
  const para = document.createElement('p');
  para.textContent = '你点击了这个按钮！';
  document.body.appendChild(para);
}
<button onclick="createParagraph()">点我呀</button>
```

这个 demo 与之前的两个功能完全一致，只是在`button`元素中包含了一个内联的 `onclick` 处理器，使得函数在按钮被按下时运行。

**然而请不要这样做。** 这将使 JavaScript 污染到 HTML，而且效率低下。对于每个需要应用 JavaScript 的按钮，你都得手动添加 `onclick="createParagraph()"` 属性。

可以使用纯 JavaScript 结构来通过一个指令选取所有按钮。下文的这段代码即实现了这一目的：

```js
const buttons = document.querySelectorAll('button');

for(let i = 0; i < buttons.length ; i++) {
  buttons[i].addEventListener('click', createParagraph);
}
```

这样写乍看去比 `onclick` 属性要长一些，但是这样写会对页面上所有按钮生效，无论多少个，或添加或删除，完全无需修改 JavaScript 代码。

**注**：请尝试修改 `apply-javascript.html` 以添加更多按钮。刷新后可发现按下任一按钮时都会创建一个段落。很高效吧。

#### `async` 和 `defer`

脚本阻塞问题实际有两种解决方案 —— `async` 和 `defer`。我们来依次讲解。

浏览器遇到 `async` 脚本时不会阻塞页面渲染，而是直接下载然后运行。这样脚本的运行次序就无法控制，只是脚本不会阻止剩余页面的显示。当页面的脚本之间彼此独立，且不依赖于本页面的其它任何脚本时，`async` 是最理想的选择。

比如，如果你的页面要加载以下三个脚本：

```html
<script async src="js/vendor/jquery.js"></script>

<script async src="js/script2.js"></script>

<script async src="js/script3.js"></script>
```

三者的调用顺序是不确定的。`jquery.js` 可能在 `script2` 和 `script3` 之前或之后调用，如果这样，后两个脚本中依赖 `jquery` 的函数将产生错误，因为脚本运行时 `jquery` 尚未加载。

解决这一问题可使用 `defer` 属性，脚本将按照在页面中出现的顺序加载和运行：

```html
<script defer src="js/vendor/jquery.js"></script>

<script defer src="js/script2.js"></script>

<script defer src="js/script3.js"></script>
```

添加 `defer` 属性的脚本将按照在页面中出现的顺序加载，因此第二个示例可确保 `jquery.js` 必定加载于 `script2.js` 和 `script3.js` 之前，同时 `script2.js` 必定加载于 `script3.js` 之前。

脚本调用策略小结：

- 如果脚本无需等待页面解析，且无依赖独立运行，那么应使用 `async`。
- 如果脚本需要等待页面解析，且依赖于其它脚本，调用这些脚本时应使用 `defer`，将关联的脚本按所需顺序置于 HTML 中。

### 比较运算符

| 运算符 | 名称                           | 示例                       |
| :----- | :----------------------------- | :------------------------- |
| `===`  | 严格等于（它们是否完全一样？） | `5 === 2 + 4`              |
| `!==`  | 不等于（它们究竟哪里不一样？） | `'Chris' !== 'Ch' + 'ris'` |
| `<`    | 小于                           | `10 < 6`                   |
| `>`    | 大于                           | `10 > 20`                  |