---
title: Git从入门到入门
date: 2016-9-11 19:43:41
categories: Android

---
## 简介

git命令是一些命令行工具的集合，它可以用来跟踪，记录文件的变动。比如你可以进行保存，比对，分析，合并等等。这个过程被称之为版本控制。已经有一系列的版本控制系统，比如SVN, Mercurial, Perforce, CVS, Bitkeepe等等。

Git是分布式的，这意味着它并不依赖于中心服务器，任何一台机器都可以有一个本地版本的控制系统，我们称之为仓库。如果是多人协作的话，你需要还需要一个线上仓库，用来同步信息。这就是GitHub, BitBucket的工作。

## 使用

本地目录下，在命令行里新建一个代码仓库（repository）
里面只有一个README.md
<!-- more -->
命令如下：	
```shell
	touch README.md
	git init
```
初始化repository
```shell
	git add README.md
```
将README.md加入到缓存区 一次只加一个  很不爽的说
```shell
	git add --a
```
将所有改动提交到缓存（注意是两个杠） 酸爽
```shell
	git commit -m "first commit"
```
提交改变，并且附上提交信息"first commit"

## Push
```shell
	git remote add origin https://github.com/XXX(username)/YYYY(projectname).git
```
加上一个remote的地址，名叫origin，地址是github上的地址（Create a new repo就会有）
因为Git是分布式的，所以可以有多个remote.
```shell
	git push -u origin master
```
将本地内容push到github上的那个地址上去。
参数-u
用了参数-u之后，以后就可以直接用不带参数的git pull从之前push到的分支来pull。
此时如果origin的master分支上有一些本地没有的提交,push会失败.
所以解决的办法是, 首先设定本地master的上游分支:
```shell
	git branch --set-upstream-to=origin/master
```
然后pull:
```shell
	git pull --rebase
```
最后再push:
```shell
	git push
```
 

## 分支
新建好的代码库有且仅有一个主分支（master），它是自动建立的。

```shell
	git branch develop master
```
新建一个叫develop的分支，基于master分支

切换到这个分支：

```shell
	git checkout develop
```
现在可以在这个develop分支上做一些改动，并且提交。
注意：
切换分支的时候可以发现，在Windows中的repository文件夹中的文件内容也会实时相应改变，变成当前分支的内容。

在本地新建一个分支： 
```shell
	git branch Branch1
```
切换到你的新分支: 	
```shell
	git checkout Branch1
```
将新分支发布在github上： 
```shell
	git push origin Branch1
```
在本地删除一个分支： 
```shell
	git branch -d Branch1
```
在github远程端删除一个分支： 
```shell
	git push origin :Branch1   (分支名前的冒号代表删除)
```

push方法1：

现在如果想直接Push这个develop分支上的内容到github
```shell
	git push -u origin
```
如果是新建分支第一次push，会提示：
```java
	fatal: The current branch develop has no upstream branch.
	To push the current branch and set the remote as upstream, use
	git push --set-upstream origin develop
```
输入这行命令，然后输入用户名和密码，就push成功了。

以后的push就只需要输入
```shell
	git push origin
```

push方法2：

比如新建了一个叫dev的分支，而github网站上还没有，可以直接：
```shell
	git push -u origin dev
```
这样一个新分支就创建好了。


push方法3：

提交到github的分支有多个，提交时可以用这样的格式：
　　
```shell
	git push -u origin local:remote
```
比如：git push -u origin master:master

表明将本地的master分支（冒号前）push到github的master分支（冒号后）。
如果左边不写为空，将会删除远程的右边分支。

## 删除分支

```shell
	git branch
```		
可以查看所有的分支

```shell
	git branch -d develop2
```
将develop2分支删除


## 远端仓库

1.链接远端仓库 - git remote add

为了能够上传到远端仓库，我们需要先建立起链接，这篇教程中，远端仓库的地址为：https://github.com/tutorialzine/awesome-project,但你应该自己在Github, BitBucket上搭建仓库，自己一步一步尝试。

添加测试用的远端仓库
```shell
	git remote add origin https://github.com/tutorialzine/awesome-project.git
```
一个项目可以同时拥有好几个远端仓库为了能够区分，通常会起不同的名字。通常主远端仓库被称为origin。


2.克隆仓库 - git clone

放在Github上的开源项目，人们可以看到你的代码。可以使用 git clone进行下载到本地。
```shell
	git clone https://github.com/tutorialzine/awesome-project.git
```
本地也会创建一个新的仓库，并自动将github上的分支设为远端分支。

 
## 加Tag
　　git tag tagname develop
　　git tag中的两个参数，一个是标签名称，另一个是希望打标签的点develop分支的末梢。

 

## 合并分支
```shell
	git checkout master
```
先转到主分支
```shell
　　git merge --no-ff develop
```
然后把develop分支merge过来
```shell
	git branch -d amazing_new_feature
```
删除掉分支
　　
参数意义：
　　不用参数的默认情况下，是执行快进式合并。
　　使用参数--no-ff，会执行正常合并，在master分支上生成一个新节点。
　　merge的时候如果遇到冲突，就手动解决，然后重新add，commit即可。
[GUI工具](https://git-scm.com/download/gui/linux)


## 高级

这篇文章的最后一节，我们来说些比较高级并且使用的技巧。

1.比对两个不同提交之间的差别

每次提交都有一个唯一id，查看所有提交和他们的id，可以使用 git log:
```shell
	git log

commit ba25c0ff30e1b2f0259157b42b9f8f5d174d80d7
Author: Tutorialzine
Date:   Mon May 30 17:15:28 2016 +0300

    New feature complete

commit b10cc1238e355c02a044ef9f9860811ff605c9b4
Author: Tutorialzine
Date:   Mon May 30 16:30:04 2016 +0300

    Added content to hello.txt

commit 09bd8cc171d7084e78e4d118a2346b7487dca059
Author: Tutorialzine
Date:   Sat May 28 17:52:14 2016 +0300

    Initial commit
```
id 很长，但是你并不需要复制整个字符串，前一小部分就够了。
查看某一次提交更新了什么，使用 git show:
```shell
	git show b10cc123

commit b10cc1238e355c02a044ef9f9860811ff605c9b4
Author: Tutorialzine
Date:   Mon May 30 16:30:04 2016 +0300

    Added content to hello.txt

diff --git a/hello.txt b/hello.txt
index e69de29..b546a21 100644
--- a/hello.txt
+++ b/hello.txt
@@ -0,0 +1 @@
+Nice weather today, isn't it?
```
查看两次提交的不同，可以使用git diff [commit-from]..[commit-to] 语法：
```shell
	git diff 09bd8cc..ba25c0ff

diff --git a/feature.txt b/feature.txt
new file mode 100644
index 0000000..e69de29
diff --git a/hello.txt b/hello.txt
index e69de29..b546a21 100644
--- a/hello.txt
+++ b/hello.txt
@@ -0,0 +1 @@
+Nice weather today, isn't it?
```
比较首次提交和最后一次提交，我们可以看到所有的更改。当然使用git difftool命令更加方便。

2.回滚某个文件到之前的版本

git 允许我们将某个特定的文件回滚到特定的提交，使用的也是 git checkout。

下面的例子，我们将hello.txt回滚到最初的状态，需要指定回滚到哪个提交，以及文件的全路径。
```shell
	git checkout 09bd8cc1 hello.txt
```
3.回滚提交

如果你发现最新的一次提交完了加某个文件，你可以通过 git commit —amend来修复，它会把最新的提交打回暂存区，并尝试重新提交。

如果是更复杂的情况，比如不是最新的提交了。那你可以使用git revert。

最新的一次提交别名也叫HEAD。
```shell
	git revert HEAD
```
其他提交可以使用id:
```shell
	git revert b10cc123
```
混滚提交时，发生冲突是非常频繁的。当文件被后面的提交修改了以后，git不能正确回滚。

4.解决合并冲突

冲突经常出现在合并分支或者是拉去别人的代码。有些时候git能自动处理冲突，但大部分需要我们手动处理。

比如John 和 Tim 分别在各自的分支上写了两部分代码。
John 喜欢 for:
```java
// Use a for loop to console.log contents.
for(var i=0; i<arr.length; i++) {
console.log(arr[i]);
}
```
Tim 喜欢 forEach:
```java
// Use forEach to console.log contents.
arr.forEach(function(item) {
console.log(item);
});
```
假设John 现在去拉取 Tim的代码:
```shell
	git merge tim_branch
Auto-merging print_array.js
CONFLICT (content): Merge conflict in print_array.js
Automatic merge failed; fix conflicts and then commit the result.
```
这时候git并不知道如何解决冲突，因为他不知道John和Tim谁写得更好。

于是它就在代码中插入标记。
```java
<<<<<<< HEAD
// Use a for loop to console.log contents.
for(var i=0; i<arr.length; i++) {
    console.log(arr[i]);
}
=======
// Use forEach to console.log contents.
arr.forEach(function(item) {
    console.log(item);
});
>>>>>>> Tim s commit.
```
==== 号上方是当前最新一次提交，下方是冲突的代码。我们需要解决这样的冲突，经过组委会成员讨论，一致认定，在座的各位都是垃圾！两个都不要。改成下面的代码。
```java
// Not using for loop or forEach.
// Use Array.toString() to console.log contents.
console.log(arr.toString());
```
好了，再提交一下：
```shell
	git add -A
	git commit -m "Array printing conflict resolved."
```
如果在大型项目中，这个过程可能容易出问题。你可以使用GUI 工具来帮助你。使用 git mergetool。

5.配置 .gitignore

大部分项目中，会有写文件，文件夹是我们不想提交的。为了防止一不小心提交，我们需要gitignore文件：

在项目根目录创建.gitignore文件
在文件中列出不需要提交的文件名，文件夹名，每个一行
.gitignore文件需要提交，就像普通文件一样
通常会被ignore的文件有：

log文件
task runner builds
node_modules等文件夹
IDEs生成的文件
个人笔记
例如：

*.log
build/
node_modules/
.idea/
my_notes.txt


