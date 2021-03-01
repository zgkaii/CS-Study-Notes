**查看WSL版本信息**

```shell
wsl --list --verbose
```

**查看Ubuntu 版本信息**

```shell
# 查看linux内核版本
uname -r

# 查看系统版本
cat /etc/lsb-release

# 查看本地磁盘内存情况（可见C盘、D盘）
df -h
```

**启用SSH**

```shell
sudo service ssh start             #启动SSH服务
sudo service ssh status            #检查状态
sudo systemctl enable ssh          #开机自动启动ssh命令
```

**zsh下载**

```shell
# 利用 apt 安装 zsh
sudo apt install zsh

# 下载安装 oh-my-zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 将 zsh 作为默认的 Shell 环境
chsh -s $(which zsh)

# 打开终端，在终端上输入: 
zsh --version
```

主题：https://www.piddnad.cn/2019/05/04/ohmyzsh/



WSL Ubuntu18里，默认映射Win C盘为`/mnt/c/`，需要手动改为`/c/`

```shell
sudo mkdir /c
sudo mount --bind /mnt/c /c
```



**WSL固定IP**

* https://github.com/MicrosoftDocs/WSL/issues/418#issuecomment-648570865

我给你一个新的主意：不要更改IP，而是添加一个指定的IP。

在Windows 10中，以管理员权限运行CMD或Powershell，然后执行以下两个命令：

```shell
# 在Ubuntu中添加IP地址192.168.50.16，名为eth0：1
wsl -d Ubuntu-16.04 -u root ip addr add 192.168.50.16/24 broadcast 192.168.50.255 dev eth0 label eth0:1

# 在Win10中添加IP地址192.168.50.88
netsh interface ip add address "vEthernet (WSL)" 192.168.50.88 255.255.255.0
```

将来，访问Ubuntu时将使用192.168.50.16，访问Win10时将使用192.168.50.88。
您可以将上述两行命令另存为.bat文件，然后将其放入引导区，并使其每次自动执行。