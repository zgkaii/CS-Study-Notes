CMD下载速度慢不是电脑问题，而是下载的网站有网速限制，如pip，虽然没被墙，但由于是外网，网速极差，经常是几KB一秒，所以我们可以采用镜像服务器，即在命令后加上：
```shell
pip install XXX -i https://pypi.tuna.tsinghua.edu.cn/simple
```

永久解决方案：
在user文件夹里新建pip文件夹，再建pip.ini 例如C:/用户/XXXX/pip/pip.ini，写入：
```shell
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple/
[install]
trusted-host=pypi.tuna.tsinghua.edu.cn
```