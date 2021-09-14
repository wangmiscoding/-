### 常用linux命令

1. cd:改变目录。
2. cd .. ：中间有空格，回退到上-一个目录,直接cd进入默认目录
3. pwd :显示当前所在的目录路径。
4. Is(ll): 都是列出当前目录中的所有文件,只不过(两个)列出的内容更为详细。
5. touch :新建一个文件 如touch index.js就会在当前目录下新建一-个index.js文件。
6. rm:删除一 个文件, rm index.js就会把index.js文件删除。
7. mkdir: 新建一-个目录,就是新建一一个文件夹。
8. rm-r:删除-一个文件夹, rm -r src删除src目录
9. mv移动文件, mv index.html src index.html是我们要移动的文件, src是目标文件夹,当然这样写，必需文件夹在同一目录下。
10. reset重新初始化终端/清屏。
11. clear清屏。
12. history查看命令历史。
13. help帮助。
14. exit退出。
15.  #表示注释

切勿尝试 rm -rf /

## Git命令

> 查看配置 git config -l

查看不同级别的配置文件 ：

#查看系统config 

git config --system --list

#查看当前用户(global)配置

git config --global --list

设置用户名，邮箱

```
git config --global user.name "wangm"
git config --global user.email 2642731099@qq.com
```

## Git基本理论

Git本地有三个工作区域  工作目录(Working Directory)、暂存区(Stage/Index) 资源库(Repository)。如果在加上远程的git仓库(Remote Directory) 就可以分为四个工作区域。文件在这四个区域之间的转换关系如下：

工作目录---git add files -->暂存区--git commit --->资源库---git push--->远程git仓库



git工作流程

1. 在工作目录中添加、修改文件；
2. 将需要进行版本管理的文件放入暂存区域；
3. 将暂存区域的文件提交到git仓库。



创建本地仓库

> git init 初始化当前项目
>
> git  clone [uri] 克隆远程项目到本地
>
> git status 查看所有文件状态
>
> git add . 添加所有文件到暂存区
>
> git commit -m  "提交消息"  提交暂存区中的文件到本地仓库



忽略文件

目录下建立 .gitignore文件，此文件有以下规则：

``` 
*.txt   忽略所有.txt文件
!lib.txt 但lib.txt除外
/temp   仅忽略项目根目录下的temp文件，不包括其他目录temp
build/  忽略build目录下的所有文件
doc/*.txt  会忽略doc/notes.txt 但不包括doc/server/arch.txt
```

设置本机绑定公钥，实现免密码登录

进入C:\users\Administrator\.ssh 目录

生成公钥,输入 ssh-keygen -t rsa



## Idea集成Git

新建项目，绑定git

将我们远程的git文件目录拷贝到项目中即可！