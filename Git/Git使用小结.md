1. Git安装

参考：https://www.liaoxuefeng.com/wiki/896043488029600/898732864121440

**提交代码至远程仓库**

2. 远程仓库

首先登录Github.com，create new repository，注意创建新项目时，全部按默认，只填写 项目 `名称` 和 `简介` ，不勾选README.md，License、Ignore；

初始化项目完成后，页面会有展示3种构建项目内容的方式；

1）从命令行创建一个新的仓库并和远程仓库关联

2）从命令行推送本地已经存在的项目到远程仓库

3）从其他仓库导入代码，例如SVN等

3. 常用命令

本地创建好的项目目录，提交至远程仓库；

1）初始化仓库

```shell
git init
```

2）将修改的文件添加至缓冲区

```shell
git add README.md
```

3）将缓冲区内容添加到当前分支

```shell
git commit -m "init repo"
```

4）重命名本地仓库分支名

```shell
git branch -M main
```

5）关联远程仓库

```shell
git remote add origin git@github.com:Lcsbs/tech-summary.git
```

6）将本地仓库内容推送至远程仓库

```shell
git push -u origin main
```

把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`main`推送到远程。

由于远程库是空的，我们第一次推送`main`分支时，加上了`-u`参数，Git不但会把本地的`main`分支内容推送的远程新的`main`分支，还会把本地的`main`分支和远程的`main`分支关联起来，在以后的推送或者拉取时就可以简化命令。

关联后，使用命令`git push -u origin main`第一次推送main分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令`git push origin main`推送最新修改；

**从远程仓库拉取代码**

1. 在电脑上安装Git并配置全局的用户名和邮箱

```shell
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

第1步：创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

生成目录是在：Your public key has been saved in /c/Users/admin/.ssh/id_rsa.pub

```shell
ssh-keygen -t rsa -C "youremail@example.com"
```

你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。

如果一切顺利的话，可以在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对，`id_rsa`是私钥，不能泄露出去，`id_rsa.pub`是公钥，可以放心地告诉任何人。

第2步：登陆GitHub，打开“Account settings”，“SSH Keys”页面：

然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容：

为什么GitHub需要SSH Key呢？因为GitHub需要识别出你推送的提交确实是你推送的，而不是别人冒充的，而Git支持SSH协议，所以，GitHub只要知道了你的公钥，就可以确认只有你自己才能推送。

当然，GitHub允许你添加多个Key。假定你有若干电脑，你一会儿在公司提交，一会儿在家里提交，只要把每台电脑的Key都添加到GitHub，就可以在每台电脑上往GitHub推送了。

最后友情提示，在GitHub上免费托管的Git仓库，任何人都可以看到喔（但只有你自己才能改）。所以，不要把敏感信息放进去。

如果你不想让别人看到Git库，有两个办法，一个是交点保护费，让GitHub把公开的仓库变成私有的，这样别人就看不见了（不可读更不可写）。另一个办法是自己动手，搭一个Git服务器，因为是你自己的Git服务器，所以别人也是看不见的。这个方法我们后面会讲到的，相当简单，公司内部开发必备。

确保你拥有一个GitHub账号后，我们就即将开始远程仓库的学习。

3. 从远程仓库拉取代码

```shell
git clone git@github.com:Lcsbs/tech-summary.git
```

