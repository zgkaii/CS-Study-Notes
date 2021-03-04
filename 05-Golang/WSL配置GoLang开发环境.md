## 自动安装

```shell
$ sudo apt install golang-go
```

但是！用这样的办法自动安装的golang并非最新版。

## 更新版本

卸载旧版本

```shell
$ apt remove golang-go
```

下载，这里选择1.15.2版本

```shell
$ wget https://studygolang.com/dl/golang/go1.15.2.linux-amd64.tar.gz
```

解压到`/usr/local`

```shell
$ tar -C /usr/local -xzf go1.15.2.linux-amd64.tar.gz
```

## 设置环境变量

/etc/profile中写入环境变量`vi /etc/profile`

```shell
export GOPATH=/home/zgkaii/go
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

`source /etc/profile`使生效。

如果`go env`查看`GOPATH`与`GOROOT`并未生效。

那么在`~/.zshrc`中最后一行加入`source /etc/priofile`或者加上跟上面一样的三条环境变量配置。

查看：

```zsh
$ go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/home/zgkaii/.cache/go-build"
GOENV="/home/zgkaii/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOINSECURE=""
GOMODCACHE="/home/zgkaii/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/home/zgkaii/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD="/home/zgkaii/go/src/github.com/zgkaii/golang-practice/go.mod"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build500707810=/tmp/go-build -gno-record-gcc-switches"
```

## 测试

```shell
# 创建文件夹
$ mkdir $GOPATH/src/github.com/zgkaii/hello -p

# 进入文件夹
$ cd $GOPATH/src/github.com/zgkaii/hello

# 创建文件hello.go
$ touch hello.go
```

hello.go:

```go
package main

import "fmt"

func main() {
 fmt.Printf("Hello, World!!!")
}
```

运行：

```shell
$ go run hello.go
hello World!!
```

## 安装go tools

### 从官方github仓库中拉取来自官方的tools

```shell
# 创建文件夹
mkdir $GOPATH/src/golang.org/x/

# 下载源码
go get -d github.com/golang/tools

# copy 
cp $GOPATH/src/github.com/golang/tools $GOPATH/src/golang.org/x/ -rf
```

执行上述三条命令后会把golang官方的tools全部拉取到本地，且以合适的文件组织方式组织。（从golang.org拉取会优先在本地$GOPATH/src/golang.org/中搜索，如果找到就不必再上[https://golang.org拉取](https://golang.xn--org-jx3ek79d/)）

### 按提示拉取非官方工具

在vscode提示安装的工具中，一些来自非官方的github仓库，此时这些工具你的本地还没有，请对应使用go get指令。

例如：

```shell
installing github.com/sqs/goreturns FAILED 
```

对应应该在终端中执行：

```shell
go get github.com/sqs/goreturns
```

> 参考：https://studygolang.com/articles/30683