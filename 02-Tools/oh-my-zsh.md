### 安装、配置

先看下自己有哪一些 shell

```shell
cat /etc/shells
```

安装 `zsh` 并将之设置为默认 Shell：

- 利用 apt 安装 `zsh`

  ```bash
  $ sudo apt install zsh
  ```

下载安装 [oh-my-zsh](https://ohmyz.sh/)，可能是市面上最好的 `zsh` 配置管理工具：

- 运行命令下载安装

  ```bash
  $ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
  ```

  ![](https://cdn.spencer.felinae98.cn/github/2020/09/200902_220651.png)

- 将 `zsh` 作为默认的 Shell 环境（如果刚刚安装脚本没有这样做的话）：

  ```bash
  $ chsh -s $(which zsh)
  ```

### 插件、主题模板推荐

`zsh` 拥有相当丰富的主题、插件等拓展内容，这里我仅列举一些我常用的配置插件和主题模板：

#### 插件

![](https://cdn.spencer.felinae98.cn/github/2020/09/200902_220651-1.png)

- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)：为 `zsh` 提供基于输入历史的自动命令提示
- [autojump](https://github.com/wting/autojump)：快速跳转不同的目录、路径、文件夹
- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)：为 `zsh` 命令提供色彩高亮

#### 主题模板

![](https://cdn.spencer.felinae98.cn/github/2020/09/200902_220651-2.png)

- [starship - The cross-shell prompt for astronauts](https://starship.rs/)
- [powerlevel10k - A fast reimplementation of Powerlevel9k ZSH theme](https://github.com/romkatv/powerlevel10k)

主题模板和插件的安装在它们各自的文档中都有详细的介绍，这里不再赘述。

#### 解决乱码

当把zsh主题换为“agnoster”后，图标出现乱码：	

```shell
sudo vim ~/.zshrc
	修改ZSH_THEME="agnoster"
source ~/.zshrc
```

这是因为得先安装 [Powerline Fonts](https://link.zhihu.com/?target=https%3A//github.com/powerline/fonts) 这样才会没有乱码，我们不能再 wsl 里面安装该字体，需要在 win10 下安装：

```shell
git clone https://github.com/powerline/fonts.git
```

切换进fonts目录，输入命令：

```cmd
.\install.ps1
```

这将在Windows上安装所有字体。再次打开WSL，发现乱码还未消失，这是需要修改WSL字体。打开Temieal的`setting.json文件`，修改fontFace（只要修改为XXX for Powerline即可），例如：

```json
        "fontFace": "Noto Mono for Powerline",
```

重写打开WSL，乱码消失了。

**解决 vscode 使用 wsl 的 乱码**

打开Font 的setting.json文件，加入：

```json
    "editor.fontFamily": "Fira Code",
    "editor.fontLigatures": true,
```

> 安装Fira Code：https://www.jianshu.com/p/2f54a5de60b0

所有乱码解决。