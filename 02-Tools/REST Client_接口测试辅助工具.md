> 文章来源——[VSCode 小鸡汤 第01期 - REST Client 简单好用的接口测试辅助工具](https://segmentfault.com/a/1190000018091951)

## 介绍

今天给大家介绍一个后端开发辅助的好工具 —— REST Client，插件如其名这就是一个 REST 的客户端插件，把我们的 VSCode 转化为一个 REST 接口测试的利器

![REST Client](https://segmentfault.com/img/remote/1460000018091954)

我们一般都会用 PostMan 来完成接口测试的工作，因为用起来十分简单快捷，但是一直以来我也在寻找更好的方案，一个不用切换窗口多开一个 app 的方案 —— 终于在使用 VSCode 一段时版本间，我找到了 REST Client 插件，初看 REST Client 插件的时候，会觉得他十分的简陋，但是在使用一段时间后会发现在 REST Client 插件中已经有完成接口测试所需的所有东西

- 优势
  - 基于 HTTP 语言，HTTP 语言是一门非常简单的语言，使用 HTTP 语言可以轻松的描述请求
  - 纯文本记录，不同于 PostMan 保存在云端，或是 Paw 那样保存二进制文件，并且纯文本可以使用 git 追踪内容的变化
  - 无需切换窗口，测试，调试，代码编辑都在一个 VSCode 中完成
- 劣势
  - 操作和使用不像 PostMan 之类的图形化工具那么直观
  - 不支持请求前后对数据进行操作的脚本，不过这个已经在作者的开发计划中

很多时候我们只是需要写完代码后手边有一个小工具可以轻松愉快的看一眼接口是否正常，那么 REST Client 就是我们的首选了

## 使用介绍

### 安装和入门

插件的安装非常简单，搜索 restclient 即可安装

![install](https://segmentfault.com/img/remote/1460000018091955?w=799&h=230)

安装完成后，可以在命令菜单中找到 REST Client 相关的功能

![菜单](https://segmentfault.com/img/remote/1460000018091956)

#### 简单请求

我们先从发送最简单的请求开始

首先需要新建一个 http 文件，创建文件时后缀为 http 即可，例如 test.http

之后输入下面的内容：

```
GET http://localhost:8000/api/v1/public/echo?msg=1345asdf HTTP/1.1
```

echo 是一个测试服务，他会返回你传入的 msg 的内容，输入完上面的

这时候请求上面会显示一个 “Send Request” 按钮，点击即可发送请求，请求完成后，插件会分割当前窗口将新的结果打开在右侧的窗口中，下图中显示了请求的所有相关信息

![Send Request](https://segmentfault.com/img/remote/1460000018091957?w=1581&h=492)

### HTTP 语言基础

#### 语言入门

HTTP 是一个非常简单的语言，入门仅需几分钟

![HTTP 语言入门](https://segmentfault.com/img/remote/1460000018091958?w=1210&h=672)

最基本的 HTTP 语言语法入门可以参看上面的内容，配合 VSCode 的自动提示功能，用起来简直不要太快

也不用担心是否记得 header 里面那些选项，想不起来的时候 `Ctrl + 空格` 调出自动提示即可

要注意的地方

1. 请求文本最后面需要有一个空行，或者一个 `#` 开头的行，建议空行，这样多个请求看起来会非常好看
2. 如果需要把 `form` 类型的参数拆分为多行，那么第二个参数开始必须以 `&` 开始（如图）
3. GET 请求也可以将参数拆分多行，每行开头必须以 `?` 或者 `&` 开始

#### 发送文件

一般来说，我们使用 `multipart/form-data` 请求方式来完成

![multipart/form-data](https://segmentfault.com/img/remote/1460000018091959?w=1007&h=563)

如图配置，REST Client 就会将文件内容填充到相应的区域完成发送

#### 保存请求结果

对于返回图片的接口在 VSCode 中是可以直接预览的，如果是 Excel 之类的二进制文件，那么这里可能会显示乱码（二进制文件）

![返回图片的接口](https://segmentfault.com/img/remote/1460000018091960)

选中相应结果页，右上角提供了保存结果的按钮

![保存结果](https://segmentfault.com/img/remote/1460000018091961?w=1240&h=252)

#### 查看请求历史

使用 `Ctrl + Alt + H`（macOS 使用 `Cmd + option + H`）查看请求历史

![请求历史](https://segmentfault.com/img/remote/1460000018091962)

#### 使用变量

变量的好处，在开发过程中我们都知道，在 HTTP 语言中同样可以使用变量来帮助我们组织请求代码

##### 自定义变量

我们可以在 http 文件中直接定义变量，使用 `@` 符号开头，以 `{{variable name}}` 的格式来使用

```
@host = http://localhost:8000
@token = adsfasdfasdfadsfasdfasdfas

### test
GET {{host}}/api/v1/public/echo HTTP/1.1
    ?msg=1345asdf
    &bundle_id=demo
    &test=1
    &token={{token}}

### test request
POST {{host}}/api/v1/public/echo HTTP/1.1
Content-Type: application/x-www-form-urlencoded
User-Agent: iPhone

test=1
&bundle_id=demo
&msg=123123
&token={{token}}
```

这样在测试验证不同环境接口正确性的场合，我们可以很方便的在不同服务器之间切换，或是所有接口都使用同一个参数的时候非常方便例如上面的 token 应该是大部分接口都会使用到的

##### 环境变量

除了使用自定义变量以外还可以对当前的项目或是创建编辑器全局的环境变量

```
"rest-client.environmentVariables": {
    "$shared": {
        "version": "v1"
    },
    "local": {
        "version": "v2",
        "host": "http://localhost:8000",
        "token": "tokentokentokentoken1"
    },
    "prod": {
        "host": "http://api.xxxxxx.com",
        "token": "tokentokentoken2"
    }
}
```

上面 `$shared` 中的变量表示在所有环境设置中都可以使用的

设置后可通过 `Ctrl + Alt + E`（`Cmd + option + E`）切换环境

![环境变量切换](https://segmentfault.com/img/remote/1460000018091963)

##### 系统变量

REST Client 提供了一些自带的系统变量，方便我们直接使用（这里由于我没有使用过 Azure 所以跳过了 Azure 相关的变量，大家可以参考文档使用）

- `{{$guid}}`: 生成一个 UUID
- `{{$randomInt min max}}`: 生成随机整数
- `{{$timestamp [offset option]}}`: 生成时间戳，可以使用类似 `{{$timestamp -3 d}}` 生成3天前的时间戳，或是使用 `{{$timestamp 2 h}}` 这样的形式生成2小时后的时间戳
- `{{$datetime rfc1123|iso8601 [offset option]}}`: 生成日期字符串

### VSCode 提供的辅助功能

VSCode 对我们使用 HTTP 语言提供了包括自动提示，Outline 代码导航功能，方便我们编写接口测试代码

#### 自动提示

![自动提示](https://segmentfault.com/img/remote/1460000018091964?w=703&h=268)

#### Outline 以及代码导航

![Outline 和代码导航](https://segmentfault.com/img/remote/1460000018091965?w=1268&h=261)

#### 验证和证书

##### Basic Auth

Basic Auth 可以使用已经 Base64 后的 `username:password`，也可以直接填入 `username` 和 `password`，也就是下面两种形式都是可以的

使用 Base64 的结果

```
POST {{host}}/api/v1/public/echo HTTP/1.1
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

使用 `username` 和 `password`

```
POST {{host}}/api/v1/public/echo HTTP/1.1
Authorization: Basic username password
```

##### Digest Auth

Digest Auth 直接填入 `username` 和 `password` 即可

```
POST {{host}}/api/v1/public/echo HTTP/1.1
Authorization: Digest username password
```

##### SSL 证书

ssl 证书在设置文件中对特定域名指定证书路径后，就可以自动生效了

```
"rest-client.certificates": {
    "localhost:8081": {
        "cert": "/Users/demo/Certificates/client.crt",
        "key": "/Users/demo/Keys/client.key"
    },
    "example.com": {
        "cert": "/Users/demo/Certificates/client.crt",
        "key": "/Users/demo/Keys/client.key"
    }
}
```

每个 host 我们可以设置下面的内容：

- cert: x509 证书路径
- key: 私钥路径
- pfx: PKCS #12 或者 PFX 证书路径
- passphrase: 证书密码（需要时设置）

#### 代码生成

曾经使用 Postman 的时候，Postman 的代码生成功能为我提供了非常多的方便，REST Client 中提供了同样的功能

选中一个请求后，点击右键选择 `Copy Request As cURL` 可以把当前的请求复制成 curl 的命令，也可以使用 `Ctrl + Alt + C`（macOS 下`Cmd + Option + C`）呼出代码生成菜单，选择需要生成的语言

![选择代码生成语言](https://segmentfault.com/img/remote/1460000018091966?w=609&h=370)

选择语言后选择具体代码调用的方式，比如 python 可以使用 `http.client` 库或者 `Requests` 库来发送请求

![Python 请求方式](https://segmentfault.com/img/remote/1460000018091967?w=612&h=170)

#### 命名请求

之前我们发送的所有请求都是匿名请求，匿名请求和命名请求的区别就是在一个 http 文件内，可以引用命名请求的请求信息和响应信息，在请求之间有依赖关系时这个功能非常有用，例如每次登录成功后其他请求都需要更新登录返回的 token，命名请求可以用过 JSONPath 或者 XPath 获取响应数据

![命名请求](https://segmentfault.com/img/remote/1460000018091968)

在响应中也会显示使用到当前命名请求的变量值的更新

![请求响应](https://segmentfault.com/img/remote/1460000018091969)

#### 一些有用的设置

##### 设置响应显示内容

在 REST Client 设置中的 “Preview Option” 可以设置请求响应显示什么内容，总共有四种，`full`，`body`，`header`，`exchang`

![设置选项](https://segmentfault.com/img/remote/1460000018091970?w=569&h=214)

我们分别来看下四种结果显示什么内容

- full：Header + Body

![full](https://segmentfault.com/img/remote/1460000018091971)

- body：只显示 Body

![body](https://segmentfault.com/img/remote/1460000018091972?w=748&h=271)

- header：只显示 Header

![header](https://segmentfault.com/img/remote/1460000018091973)

- exchange：显示请求 + Header + Body

![exchange](https://segmentfault.com/img/remote/1460000018091974?w=786&h=499)

##### 其他常用的设置选项

- `rest-client.timeoutinmilliseconds`: 设置请求超时，单位毫秒
- `rest-client.showResponseInDifferentTab`: 每个响应请求创建一个新的 tab，为 false 时，每次请求会覆盖上一次的请求结果，设置为 true 时每次请求都会打开一个新的 tab，方便对比多次请求结果
- `rest-client.previewColumn`: 请求结果显示，`current` 表示显示在当前的编辑器分组 `beside` 表示显示在侧面编辑器分组（这个侧面根据编辑器的 `workbench.editor.openSideBySideDirection` 选项会显示在右面或是下面
- 代理：使用`http.proxy` 和 `http.proxyStrictSSL`

## 最后

其实 Postman 和 Paw 都提供更为强力的辅助工具，这里使用 REST Client 单纯觉得 Postman 和 Paw 大部分功能我其实都用不到，因为仅仅验证接口是否正常，业务是否能跑通，所以一直在寻找一个简单的工具，REST Client 刚好满足了我所有的需求。

这里附上 REST Client 项目地址，里面也有对应的文档 https://github.com/Huachao/vscode-restclient。