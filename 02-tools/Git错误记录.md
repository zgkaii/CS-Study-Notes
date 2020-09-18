### 1、error: failed to push some refs to 'git@github.com:XXXX/XXXX'
```shell script
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
#####解决方法：
git pull --rebase origin master  
git push origin master  

### 2、Your local changes to the following files would be overwritten by merge
```shell script
error: Your local changes to the following files would be overwritten by merge:
        protected/config/main.php
Please, commit your changes or stash them before you can merge.
```
#####解决方法：
如果希望保留生产服务器上所做的改动,仅仅并入新配置项, 处理方法如下:  
git stash  
git pull  
git stash pop  
然后可以使用git diff -w +文件名 来确认代码自动合并的情况.   

反过来,如果希望用代码库中的文件完全覆盖本地工作版本. 方法如下:  
git reset --hard  
git pull  
其中git reset是针对版本,如果想针对文件回退本地修改,使用  
git checkout HEAD file/to/restore  

### 3、Pull is not possible because you have unmerged files
```shell script
Pull is not possible because you have unmerged files.
Please, fix them up in the work tree, and then use 'git add/rm <file>'
as appropriate to mark resolution, or use 'git commit -a'.
```
#####解决方法：
查看冲突文件  
1、git status也可以告诉我们冲突的文件；   
Unmerged paths:  
  (use "git add <file>..." to mark resolution)  

        both modified:      file  
2、手动解决冲突，然后提交更改：  
vi file  
git add file  
git commit -m '解决冲突'  

### 4、CONFLICT (content):Merge conflict in readme.txt
```shell script
$ git merge feature1
Auto-merging readme.txt
CONFLICT (content):Merge conflict in readme.txt
Automatic merge failed; fix conflicts andthen commit the result.
```
#####解决方法：
1、git status也可以告诉我们冲突的文件；  
#       both modified:      readme.txt  #冲突文件为readme.txt  
2、查看readme.txt的内容：  
<<<<<<<HEADmaster  
=======fenzhi  

>feature1  
Git用<<<<<<<，=======，>>>>>>>标记出不同分支的内容，我们修改如下后保存：  
master and fenzhi  
3、再提交：   
$ git add readme.txt   
$ git commit -m "hebing"  


### 5、The branch 'feature-vulcan' is not fully merged.
```shell script
error:The branch 'feature-vulcan' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature-vulcan'.
```
#####解决方法：
强行删除，需要使用命令：  
git branch -D feature-vulcan   


### 6、Please move or remove them before you can merge.  Aborting
```shell script
error:The branch 'feature-vulcan' is not fully merged.
If you are sure you want to delete it, run 'git branch -D feature-vulcan'.
```
#####解决方法：
git clean  -d  -fx ""  
其中  
x  -----删除忽略文件已经对git来说不识别的文件  
d  -----删除未被添加到git的路径中的文件  
f  -----强制运行   


[Git错误总结](https://blog.51cto.com/qiangsh/1769956)