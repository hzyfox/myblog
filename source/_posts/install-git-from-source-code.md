---
title: install git from source code
date: 2017-04-26 20:14:14
tags:
- git
- centos
- "source code"
categories:
- git
---
centos下 git从源码下载到编译到配置
<!-- more -->

# git从源码编译到配置环境
大家在使用centos系统做服务器的时候，一定发现了，centos为了系统的稳定性，软件的版本都特别老。本人最近想用git把本地的project push到远程VPS的no bare仓库中，但是发现我VPS的git的receive.denyCurrentBranch竟然不支持updateInstead选项。查看一下版本才发现VPS的git版本还是V1.8.0。于是有了自己从源码编译git的想法，在这期间遇到了一些恨奇怪的问题，在此与大家分享。

## 软件环境
> - 操作系统：centos7.2

## 需要安装的库
若是条件允许，从源代码安装有很多好处，至少可以安装最新的版本。Git 的每个版本都在不断尝试改进用户体验，所以能通过源代码自己编译安装最新版本就再好不过了。有些 Linux 版本自带的安装包更新起来并不及时，所以除非你在用最新的 distro 或者 backports，那么从源代码安装其实该算是最佳选择。
Git 的工作需要调用 curl，zlib，openssl，expat，libiconv 等库的代码，所以需要先安装这些依赖工具。在有 yum 的系统上（比如 Fedora）或者有 apt-get 的系统上（比如 Debian 体系），可以用下面的命令安装：

- 对于yum用户 `sudo yum yum -y install gcc openssl openssl-devel curl curl-devel unzip perl perl-devel expat expat-devel zlib zlib-devel asciidoc xmlto gettext-devel openssh-clients`

## 源码准备
git的源码隐藏的比较深，我翻了很久才找到下载资源的官网链接 从下面链接中选取合适的版本后复制对应版本的链接 使用wget 命令下载用户目录。
> [git offical source code](https://www.kernel.org/pub/software/scm/git/)

## 编译过程
当需要的库都准备完毕后
```shell
 tar -zxf git-<your-version>.tar.gz
 cd git-<your-version>
 make prefix=/usr/local all
 sudo make prefix=/usr/local install
```
## 环境变量配置
```shell
vi ~/.bashrc
PATH=/usr/local/bin:${PATH}
source ~/.bashrc
git --vsersion
```
如果没有出现问题的话，`git --version`就可以看到git 的版本了

> note: prefix中指定过的目录中会生成 lib64/ bin/ libexec/ share/四个文件夹， 切记不要去移动这四个文件夹 否则会发生不可预知的错误。
>  我猜测是因为编译的时候指定了链接的位置，所以移动文件夹会出现不可预知的错误
> 如git 的alias无法使用 Expansion of alias 'st' failed; 'status' is not a command ![git-alias-error](git-alias-error.png)

## git常用个人设置
### bash界面显示分支名
当开发的工程有很多分支时，常常会忘记了自己在哪个分支，于是我们通过在~/.bashrc中采取一些设置即可实现下图效果
![git-branch-prompt](git-branch-prompt.png)
将下面代码复制到~/.bashrc中 并 source ~/.bashrc 即可实现上述效果:
{% codeblock lang:shell %}
find_git_branch () {
  local dir=. head
  until [ "$dir" -ef / ]; do
    if [ -f "$dir/.git/HEAD" ]; then
      head=$(< "$dir/.git/HEAD")
      if [[ $head = ref:\ refs/heads/* ]]; then
        git_branch=" → ${head#*/*/}"
      elif [[ $head != '' ]]; then
        git_branch=" → (detached)"
      else
        git_branch=" → (unknow)"
      fi
      return
    fi
    dir="../$dir"
  done
  git_branch=''
}

PROMPT_COMMAND="find_git_branch; $PROMPT_COMMAND"
# Heree

black=$'\[\e[1;30m\]'

red=$'\[\e[1;31m\]'

green=$'\[\e[1;32m\]'

yellow=$'\[\e[1;33m\]'

blue=$'\[\e[1;34m\]'

magenta=$'\[\e[1;35m\]'

cyan=$'\[\e[1;36m\]'

white=$'\[\e[1;37m\]'

normal=$'\[\e[m\]'

PS1="$white[$magenta\u$white@$green\h$white:$cyan\w$yellow\$git_branch$white]\$ $normal"
{% endcodeblock %}

### git设置difftool和mergetool

> {% codeblock 在mac的~/.gitconfig下 lang:shell  %}
[diff]
  tool = meld
[difftool]
  prompt = false
[difftool "meld"]
  trustExitCode = true
  cmd = open -W -a Meld --args \"$LOCAL\" \"$PWD/$REMOTE\"
[merge]
  tool = meld
[mergetool]
  prompt = false
[mergetool "meld"]
  trustExitCode = true
  cmd = open -W -a Meld --args --auto-merge \"$PWD/$LOCAL\" \"$PWD/$BASE\" \"$PWD/$REMOTE\" --output=\"$PWD/$MERGED\"
{% endcodeblock %}

> {% codeblock 在linux的~/.gitconfig下 lang:shell  %}
# Add the following to your .gitconfig file.
[diff]
    tool = meld
[difftool]
    prompt = false
[difftool "meld"]
    cmd = meld "$LOCAL" "$REMOTE"
# Add the following to your .gitconfig file.
[merge]
    tool = meld
[mergetool "meld"]
    # Choose one of these 2 lines (not both!) explained below.
    #cmd = meld "$LOCAL" "$MERGED" "$REMOTE" --output "$MERGED"
    cmd = meld "$LOCAL" "$BASE" "$REMOTE" --output "$MERGED"
{% endcodeblock %}
> note: 强烈建议用户使用meld前先查看stackoverflow中的一个帖子 [Setting up and using Meld as your git difftool and mergetool](http://stackoverflow.com/questions/34119866/setting-up-and-using-meld-as-your-git-difftool-and-mergetool)
### git 别名设置
{% codeblock 在linux的~/.gitconfig下 lang:shell  %}
[alias]
  last = log -1
  co = checkout
  st = status
  mt = mergetool
  dt = difftool
  lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
  ps = push server master
  br = branch
  ci = commit
{% endcodeblock %}
### git commit模板
> 一个好的用户提交习惯对于开发过程中的debug是非常重要的，不仅可以更清楚自己的开发进度，也可以在项目出问题的时候更快的找到出问题的地方是在哪一次提交之后。并且在多人合作的开发中统一记录模板是十分有必要的
> 我的一个git commit 模板如下图所示
> ![commit 模板](git-commit-templete.png)
> feature: 本次提交是一个功能的开发
> fix: 表示本次提交是一个bug的修复
> debug: 也可以加一个debug标题，代表是对bug的尝试性修复，但是不知bug是已被完善，还需进一步测试
> overview: 总结这次提交的信息
> Deatiled Description: 详细信息分条列序
> 同时项目是否在本次提交中 build 通过， functionaltest(功能测试)是否通过 performancetest(性能测试)是否通过
> {% codeblock  lang:shell  %}
(Fix/Feature/Debug):

Overview:

Detailed Description:
        1.
        2.

Build:
FunctionalTest:
PerformanceTest:
{% endcodeblock %}
> **git设置模板**
> 1. 在用户根目录下创建 文件 .commit_template 复制上面内容
> 2. 编辑~/.gitconfig文件，添加下面的内容
>{% codeblock 在linux的~/.gitconfig下 lang:shell  %}
[commit]
        template = /Users/husterfox/.commit_template
{% endcodeblock %}


## 参考
> - [安装git](https://git-scm.com/book/zh/v1/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
> - 特别感谢 **堂Di** [在Shell提示符中显示Git分支名称](http://www.cnblogs.com/cuoreqzt/p/5848224.html)
> - 强烈建议用户使用前meld前先查看stackoverflow中的一个帖子 [Setting up and using Meld as your git difftool and mergetool](http://stackoverflow.com/questions/34119866/setting-up-and-using-meld-as-your-git-difftool-and-mergetool)
> - 更详细的git教程 可以关注[廖雪峰老师的git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b00)




