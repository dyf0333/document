# linux svn 用钩子 hooks 服务器自动更新最新代码

## 1.搭建svn

安装采用YUM一键安装：

```
yum -y install subversion
```

建立版本库目录
     
     mkdir /www/svndata
     svnserve -d -r /www/svndata

建立版本库，创建一个新的Subversion项目

     svnadmin create /www/svndata/project

配置权限，conf目录下就是我们需要配置的三个文件：
          
          authz  passwd  svnserve.conf

配置账号密码

    svnserve.conf,打开下面这条语句的注释就行
    password-db = passwd

打开passwd文件，加入一个用户并制定密码即可
    
    test = 123456


开启svn服务

    svnserve -r -T -d /www/svndata

这个命令加入到开机启动：
    
    vi /etc/rc.local


注意点：

```
防火墙开启：
     3690端口
腾讯云有可能在 控制台 控制端口，要去那里配置（坑。。。）
```

## 2.配置hooks

>svn目录 ：/usr/local/svndata/test

>项目目录：/usr/local/nginx/html/test


cd /usr/local/nginx/html/test 

进入项目目录把项目checkout出来

     cd /usr/local/nginx/html
     svn checkout svn://localhost/php

```
1，cd  /usr/local/svndata/test/hooks/
2，cp cp post-commit.tmpl post-commit (复制这份模板文件，因为svn将要执行的是post-commit文件)
3，vim post-commit
```
将最后面几行删除
```
REPOS="$1"
REV="$2"
mailer.py commit "$REPOS" "$REV" /path/to/mailer.conf
```

然后加上自己将要执行的同步的命令

```
export LANG=zh_CN.UTF-8  #(这句话比较重要,如果客户端跟服务器编码不一样会出现同步失败)
WEB=/opt/lampp/test  #(将要同步过去的web项目路径)
/usr/bin/svn update $WEB  #(/usr/bin/svn代表你的svn服务文件地址 如果是通过yum安装的话，或者已经注册了svn服务，则可以直接使用svn，不需要输入全路径)
wq!保存退出，此时已经完成更新命令
```
设置post-commit文件可以执行权限(若不设置则会出现commit false 255错误)
chmod  a+x  post-commit  (或者chmod  777 post-commit)


## 3.测试

在本地电脑checkout 项目

改变

commit

查看服务器上的代码是否改变

done!
