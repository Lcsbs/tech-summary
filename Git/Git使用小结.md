1. Git安装

参考：https://www.liaoxuefeng.com/wiki/896043488029600/898732864121440

2. 远程仓库

首先登录Github.com，create new repository，注意创建新项目时，全部按默认，只填写 项目 *名称* 和 *简介* ，不勾选README.md，License、Ignore；

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