<!-- MarkdownTOC -->
- [关机/重启/注销](#关机重启注销)
- [系统信息和性能查看](#系统信息和性能查看)
- [磁盘和分区](#磁盘和分区)
- [用户和用户组](#用户和用户组)
- [网络和进程管理](#网络和进程管理)
- [常见系统服务命令](#常见系统服务命令)
- [文件和目录操作](#文件和目录操作)
- [文件查看和处理](#文件查看和处理)
- [打包和解压](#打包和解压)
- [RPM包管理命令](#rpm包管理命令)
- [YUM包管理命令](#yum包管理命令)
- [DPKG包管理命令](#dpkg包管理命令)
- [APT软件工具](#apt软件工具)

<!-- /MarkdownTOC -->

> 文章来源：公众号《Code Sheep》

## 关机/重启/注销

| 常用命令          | 作用                     |
| ----------------- | ------------------------ |
| shutdown -h now   | shutdown -h now          |
| shutdown -h 10    | 10分钟后关机             |
| shutdown -h 11:00 | 11：00关机               |
| shutdown -h +10   | 预定时间关机（10分钟后） |
| shutdown -c       | 取消指定时间关机         |
| shutdown -r now   | 重启                     |
| shutdown -r 10    | 10分钟之后重启           |
| shutdown -r 11:00 | 11：00定时重启           |
| reboot            | 重启                     |
| init 6            | 重启                     |
| init 0            | ⽴刻关机                  |
| telinit 0         | 关机                     |
| poweroff          | ⽴刻关机                  |
| halt              | 关机                     |
| sync              | buff数据同步到磁盘       |
| logout            | 退出登录Shell            |

## 系统信息和性能查看

| 常用命令                    | 作用                              |
| --------------------------- | --------------------------------- |
| uname -a                    | 查看内核/OS/CPU信息               |
| uname -r                    | 查看内核版本                      |
| uname -m                    | 查看处理器架构                    |
| arch                        | 查看处理器架构                    |
| hostname                    | 查看计算机名                      |
| who                         | 显示当前登录系统的用户            |
| who am i                    | 显示登录时的用户名                |
| whoami                      | 显示当前用户名                    |
| cat /proc/version           | 查看linux版本信息                 |
| cat /proc/cpuinfo           | 查看CPU信息                       |
| cat /proc/interrupts        | 查看中断                          |
| cat /proc/loadavg           | 查看系统负载                      |
| uptime                      | 查看系统运⾏时间、用户数、负载     |
| env                         | 查看系统的环境变量                |
| lsusb -tv                   | 查看系统USB设备信息               |
| lspci -tv                   | 查看系统PCI设备信息               |
| lsmod                       | 查看已加载的系统模块              |
| grep MemTotal /proc/meminfo | 查看内存总量                      |
| grep MemFree /proc/meminfo  | 查看空闲内存量                    |
| free -m                     | 查看内存用量和交换区用量          |
| date                        | 显示系统⽇期时间                   |
| cal 2021                    | 显示2021⽇历表                     |
| top                         | 动态显示cpu/内存/进程等情况       |
| vmstat 1 20                 | 每1秒采⼀次系统状态，采20次        |
| iostat                      | 查看io读写/cpu使用情况            |
| sar -u 1 10                 | 查询cpu使用情况（1秒⼀次，共10次） |
| sar -d 1 10                 | 查询磁盘性能                      |

## 磁盘和分区

| 常用命令                            | 作用                           |
| ----------------------------------- | ------------------------------ |
| fdisk -l                            | 查看所有磁盘分区               |
| swapon -s                           | 查看所有交换分区               |
| df -h                               | 查看磁盘使用情况及挂载点       |
| df -hl                              | 同上                           |
| du -sh /dir                         | 查看指定某个目录的⼤⼩           |
| du -sk * \|sort -rn                 | 从⾼到低依次显示文件和目录⼤⼩    |
| mount /dev/hda2 /mnt/hda2           | 挂载hda2盘                     |
| mount -t ntfs /dev/sdc1 /mnt/usbhd1 | 指定文件系统类型挂载（如ntfs） |
| mount -o loop xxx.iso /mnt/cdrom    | 挂载iso文件                    |
| mount /dev/sda1 /mnt/usbdisk        | 挂载usb盘/闪存设备             |
| umount -v /dev/sda1                 | 通过设备名卸载                 |
| umount -v /mnt/mymnt                | 通过挂载点卸载                 |
| fuser -km /mnt/hda1                 | 强制卸载(慎用)                 |

## 用户和用户组

| 常用命令                                              | 作用                                           |
| ----------------------------------------------------- | ---------------------------------------------- |
| useradd codesheep                                     | 创建用户                                       |
| userdel -r codesheep                                  | 删除用户                                       |
| usermod -g group_name user_name                       | 修改用户的组                                   |
| usermod -aG group_name user_name                      | 将用户添加到组                                 |
| usermod -s /bin/ksh -d /home/codepig –g dev codesheep | 修改用户codesheep的登录Shell、主目录以及用户组 |
| groups test                                           | 查看test用户所在的组                           |
| groupadd group_name                                   | 创建用户组                                     |
| groupdel group_name                                   | 删除用户组                                     |
| groupmod -n new_name old_name                         | 重命名用户组                                   |
| su - user_name                                        | 完整切换到⼀个用户环境                          |
| passwd                                                | 修改⼝令                                        |
| passwd codesheep                                      | 修改某用户的⼝令                                |
| w                                                     | 查看活动用户                                   |
| id codesheep                                          | 查看指定用户codesheep信息                      |
| last                                                  | 查看用户登录⽇志                                |
| crontab -l                                            | 查看当前用户的计划任务                         |
| cut -d: -f1 /etc/passwd                               | 查看系统所有用户                               |
| cut -d: -f1 /etc/group                                | 查看系统所有组                                 |

## 网络和进程管理

| 常用命令                                                      | 作用                               |
| ------------------------------------------------------------- | ---------------------------------- |
| ifconfig                                                      | 查看网络接⼝属性                    |
| ifconfig eth0                                                 | 查看某网卡的配置                   |
| route -n                                                      | 查看路由表                         |
| netstat -lntp                                                 | 查看所有监听端⼝                    |
| netstat -antp                                                 | 查看已经建⽴的TCP连接               |
| netstat -lutp                                                 | 查看TCP/UDP的状态信息              |
| ifup eth0                                                     | 启用eth0网络设备                   |
| ifdown eth0                                                   | 禁用eth0网络设备                   |
| iptables -L                                                   | 查看iptables规则                   |
| ifconfig eth0 192.168.1.1 netmask 255.255.255.0               | 配置ip地址                         |
| dhclient eth0                                                 | 以dhcp模式启用eth0                 |
| route add -net 0/0 gw Gateway_IP                              | 配置默认网关                       |
| route add -net 192.168.0.0 netmask 255.255.0.0 gw 192.168.1.1 | 配置静态路由到达网络192.168.0.0/16 |
| route del 0/0 gw Gateway_IP                                   | 删除静态路由                       |
| hostname                                                      | 查看主机名                         |
| host www.codesheep.cn                                         | 解析主机名                         |
| nslookup www.codesheep.cn                                     | 查询DNS记录，查看域名解析是否正常  |
| ps -ef                                                        | 查看所有进程                       |
| ps -ef \|grep codesheep                                       | 过滤出你需要的进程                 |
| kill -s name kill                                             | 指定名称的进程                     |
| kill -s pid  kill                                             | 指定pid的进程                      |
| top                                                           | 实时显示进程状态                   |
| vmstat 1 20                                                   | 每1秒采⼀次系统状态，采20次         |
| iostat                                                        | 查看io读写/cpu使用情况             |
| sar -u 1 10                                                   | 查询cpu使用情况（1秒⼀次，共10次）  |
| sar -d 1 10                                                   | 查询磁盘性能                       |

## 常见系统服务命令

| 常用命令                   | 作用         |
| -------------------------- | ------------ |
| chkconfig --list           | 列出系统服务 |
| service <服务名> status    | 查看某个服务 |
| service <服务名> start     | 启动某个服务 |
| service <服务名> stop      | 终⽌某个服务  |
| service <服务名> restart   | 重启某个服务 |
| systemctl status <服务名>  | 查看某个服务 |
| systemctl start <服务名>   | 启动某个服务 |
| systemctl stop <服务名>    | 终⽌某个服务  |
| systemctl restart <服务名> | 重启某个服务 |
| systemctl enable <服务名>  | 开启⾃启动    |
| systemctl disable <服务名> | 关闭⾃启动    |

## 文件和目录操作

| 常用命令                 | 作用                                                       |
| ------------------------ | ---------------------------------------------------------- |
| cd <目录名>              | 进⼊某个目录                                                |
| cd ..                    | 回上级目录                                                 |
| cd ../..                 | 回上两级目录                                               |
| cd                       | 进个⼈主目录                                                |
| cd -                     | 回上⼀步所在目录                                            |
| pwd                      | 显示当前路径                                               |
| ls                       | 查看文件目录列表                                           |
| ls -F                    | 查看目录中内容（显示是文件还是目录）                       |
| ls -l                    | 查看文件和目录的详情列表                                   |
| ls -a                    | 查看隐藏文件                                               |
| ls -lh                   | 查看文件和目录的详情列表（增强文件⼤⼩易读性）               |
| ls -lSr                  | 查看文件和目录列表（以文件⼤⼩升序查看）                     |
| tree                     | 查看文件和目录的树形结构                                   |
| mkdir <目录名>           | 创建目录                                                   |
| mkdir dir1 dir2          | 同时创建两个目录                                           |
| mkdir -p /tmp/dir1/dir2  | 创建目录树                                                 |
| rm -f file1              | 删除'file1'文件                                            |
| rmdir dir1               | 删除'dir1'目录                                             |
| rm -rf dir1              | 删除'dir1'目录和其内容                                     |
| rm -rf dir1 dir2         | 同时删除两个目录及其内容                                   |
| mv old_dir new_dir       | 重命名/移动目录                                            |
| cp file1 file2           | 复制文件                                                   |
| cp dir/* .               | 复制某目录下的所有文件⾄当前目录                            |
| cp -a dir1 dir2          | 复制目录                                                   |
| cp -a /tmp/dir1 .        | 复制⼀个目录⾄当前目录                                       |
| ln -s file1 link1        | 创建指向文件/目录的软链接                                  |
| ln file1 lnk1            | 创建指向文件/目录的物理链接                                |
| find / -name file1       | 从跟目录开始搜索文件/目录                                  |
| find / -user user1       | 搜索用户user1的文件/目录                                   |
| find /dir -name *.bin    | 在目录/dir中搜带有.bin后缀的文件                           |
| locate <关键词>          | 快速定位文件                                               |
| locate *.mp4             | 寻找.mp4结尾的文件                                         |
| whereis <关键词>         | 显示某⼆进制文件/可执⾏文件的路径                            |
| which <关键词>           | 查找系统目录下某的⼆进制文件                                |
| chmod ugo+rwx dir1       | 设置目录所有者(u)、群组(g)及其他⼈(o)的读(r)写(w)执⾏(x)权限 |
| chmod go-rwx dir1        | 移除群组(g)与其他⼈(o)对目录的读写执⾏权限                   |
| chown user1 file1        | 改变文件的所有者属性                                       |
| chown -R user1 dir1      | 改变目录的所有者属性                                       |
| chgrp group1 file1       | 改变文件群组                                               |
| chown user1:group1 file1 | 改变文件的所有⼈和群组                                      |

## 文件查看和处理

| 常用命令 作用                 |                                         |
| ----------------------------- | --------------------------------------- |
| cat file1                     | 查看文件内容                            |
| cat -n file1                  | 查看内容并标示⾏数                       |
| cat xxx.txt                   | awk 'NR%2==1'                           |
| tac file1                     | 从最后⼀⾏开始反看文件内容                |
| more file1                    | 查看⼀个⻓文件的内容                      |
| less file1                    | 类似more命令，但允许反向操作            |
| head -2 file1                 | 查看文件前两⾏                           |
| tail -2 file1                 | 查看文件后两⾏                           |
| tail -f /log/msg              | 实时查看添加到文件中的内容              |
| grep codesheep hello.txt      | 在文件hello.txt中查找关键词codesheep    |
| grep ^sheep hello.txt         | 在文件hello.txt中查找以sheep开头的内容  |
| grep [0-9] hello.txt          | 选择hello.txt文件中所有包含数字的⾏      |
| sed 's/s1/s2/g' hello.txt     | 将hello.txt文件中的s1替换成s2           |
| sed '/^$/d' hello.txt         | 从hello.txt文件中删除所有空⽩⾏           |
| sed '/ *#/d; /^$/d' hello.txt | 从hello.txt文件中删除所有注释和空⽩⾏     |
| sed -e '1d' hello.txt         | 从文件hello.txt 中排除第⼀⾏              |
| sed -n '/s1/p' hello.txt      | 查看只包含关键词"s1"的⾏                 |
| sed -e 's/ *$//' hello.txt    | 删除每⼀⾏最后的空⽩字符                   |
| sed -e 's/s1//g' hello.txt    | 从文档中只删除词汇s1并保留剩余全部      |
| sed -n '1,5p;5q' hello.txt    | 查看从第⼀⾏到第5⾏内容                    |
| sed -n '5p;5q' hello.txt      | 查看第5⾏                                |
| paste file1 file2             | 合并两个文件或两栏的内容                |
| paste -d '+' file1 file2      | 合并两个文件或两栏的内容，中间用"+"区分 |
| sort file1 file2              | 排序两个文件的内容                      |
| sort file1 file2              | uniq                                    |
| sort file1 file2              | uniq -u                                 |
| sort file1 file2              | uniq -d                                 |
| comm -1 file1 file2           | 比较两个文件的内容(去除'file1'所含内容) |
| comm -2 file1 file2           | 比较两个文件的内容(去除'file2'所含内容) |
| comm -3 file1 file2           | 比较两个文件的内容(去除两文件共有部分)  |

## 打包和解压

| 常用命令 作用                     |                          |
| --------------------------------- | ------------------------ |
| zip xxx.zip file                  | 压缩⾄zip包               |
| zip -r xxx.zip file1 file2 dir1   | 将多个文件+目录压成zip包 |
| unzip xxx.zip                     | 解压zip包                |
| tar -cvf xxx.tar file             | 创建⾮压缩tar包           |
| tar -cvf xxx.tar file1 file2 dir1 | 将多个文件+目录打tar包   |
| tar -tf xxx.tar                   | 查看tar包的内容          |
| tar -xvf xxx.tar                  | 解压tar包                |
| tar -xvf xxx.tar -C /dir          | 将tar包解压⾄指定目录     |
| tar -cvfj xxx.tar.bz2 dir         | 创建bz2压缩包            |
| tar -jxvf xxx.tar.bz2             | 解压bz2压缩包            |
| tar -cvfz xxx.tar.gz dir          | 创建gzip压缩包           |
| tar -zxvf xxx.tar.gz              | 解压gzip压缩包           |
| bunzip2 xxx.bz2                   | 解压bz2压缩包            |
| bzip2 filename                    | 压缩文件                 |
| gunzip xxx.gz                     | 解压gzip压缩包           |
| gzip filename                     | 压缩文件                 |
| gzip -9 filename                  | 最⼤程度压缩              |

## RPM包管理命令

| 常用命令                  | 作用                          |
| ------------------------- | ----------------------------- |
| rpm -qa                   | 查看已安装的rpm包             |
| rpm -q pkg_name           | 查询某个rpm包                 |
| rpm -q --whatprovides xxx | 显示xxx功能是由哪个包提供的   |
| rpm -q --whatrequires xxx | 显示xxx功能被哪个程序包依赖的 |
| rpm -q --changelog xxx    | 显示xxx包的更改记录           |
| rpm -qi pkg_name          | 查看⼀个包的详细信息           |
| rpm -qi pkg_name          | 查看⼀个包的详细信息           |
| rpm -qd pkg_name          | 查询⼀个包所提供的文档         |
| rpm -qc pkg_name          | 查看已安装rpm包提供的配置文件 |
| rpm -ql pkg_name          | 查看⼀个包安装了哪些文件       |
| rpm -qf filename          | 查看某个文件属于哪个包        |
| rpm -qR pkg_name          | 查询包的依赖关系              |
| pm -ivh --test xxx.rpm    | 测试安装rpm包                 |
| rpm -ivh --nodeps xxx.rpm | 安装rpm包时忽略依赖关系       |
| rpm -e xxx                | 卸载程序包                    |
| rpm -Fvh pkg_name         | 升级确定已安装的rpm包         |
| rpm -Uvh pkg_name         | 升级rpm包(若未安装则会安装)   |
| rpm -V pkg_name           | RPM包详细信息校验             |

## YUM包管理命令

| 常用命令                            | 作用                 |
| ----------------------------------- | -------------------- |
| yum repolist enabled                | 显示可用的源仓库     |
| yum search pkg_name                 | 搜索软件包           |
| yum install pkg_name                | 下载并安装软件包     |
| yum install --downloadonly pkg_name | 只下载不安装         |
| yum list                            | 显示所有程序包       |
| yum list installed                  | 查看当前系统已安装包 |
| yum list updates                    | 查看可以更新的包列表 |
| yum check-update                    | 查看可升级的软件包   |
| yum update                          | 更新所有软件包       |
| yum update pkg_name                 | 升级指定软件包       |
| yum deplist pkg_name                | 列出软件包依赖关系   |
| yum remove pkg_name                 | 删除软件包           |
| yum clean all                       | 清除缓存             |
| yum clean packages                  | 清除缓存的软件包     |
| yum clean headers                   | 清除缓存的header     |

## DPKG包管理命令

| 常用命令               | 作用                                 |
| ---------------------- | ------------------------------------ |
| dpkg -c xxx.deb        | 列出deb包的内容                      |
| dpkg -i xxx.deb        | 安装/更新deb包                       |
| dpkg -r pkg_name       | 移除deb包                            |
| dpkg -P pkg_name       | 移除deb包(不保留配置)                |
| dpkg -l                | 查看系统中已安装deb包                |
| dpkg -l pkg_name       | 显示包的⼤致信息                      |
| dpkg -L pkg_name       | 查看deb包安装的文件                  |
| dpkg -s pkg_name       | 查看包的详细信息                     |
| dpkg –unpack xxx.deb x | dpkg –unpack xxx.deb 解开deb包的内容 |

## APT软件工具

| 常用命令                  | 作用                   |
| ------------------------- | ---------------------- |
| apt-cache search pkg_name | 搜索程序包             |
| apt-cache show pkg_name   | 获取包的概览信息       |
| apt-get install pkg_name  | 安装/升级软件包        |
| apt-get purge pkg_name    | 卸载软件（包括配置）   |
| apt-get remove pkg_name   | 卸载软件（不包括配置） |
| apt-get update            | 更新包索引信息         |
| apt-get upgrade           | 更新已安装软件包       |
| apt-get clean             | 清理缓存               |
