# centos升级git

像[centos7](https://so.csdn.net/so/search?q=centos7&spm=1001.2101.3001.7020).5一般自带的git都是1.8.3.1版本的，比较老了，所以有时候需要升级一下git版本

安装依赖软件：

```
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc gcc perl-ExtUtils-MakeMaker
```

卸载系统自带的低版本:

```
yum remove git
```

下载git源码并解压

```
# 在usr/pro_repo下
cd /usr/pro_repo
wget https://github.com/git/git/archive/v2.3.0.zip
unzip v2.3.0.zip
# 创建一个文件夹，用来安装git，目录看自己常放哪个，我是放在/usr下的
mkdir /usr/local/git
cd git-2.3.0
# 配置参数，参数中的路径就是上面步骤创建的那个文件夹的路径
make prefix=/usr/local/git all
make prefix=/usr/local/git install
# 设置环境变量
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
source /etc/profile
git version
```



# git

> ref:
>
> https://www.cnblogs.com/zsmynl/articles/12876071.html
>
> https://www.cnblogs.com/zsmynl/articles/12876071.html

![image-20220310130018846](http://myimg.go2flare.xyz/img/image-20220310130018846.png)

基本概念：

- 工作区：为开发环境中的代码

- 暂存区：stage / index，一般存放在.git目录下的index文件（.git/index)，也有将暂存区称为索引（index）
- 版本库：.git就是git的版本库

## 使用远程代码新建（本地有可跳过）

```
git clone $远程仓库
```

## 一般性使用命令

```
#add将工作区的文件加入暂存区（stage）
git add .
#将暂存区的修改提交到当前分支
git commit -m ""
#添加远程仓库
git remote add $远程仓库
git remote -v

#重命名当前分支
git branch -M main

#当前暂存区状态
git status

#所有分支（远程+本地）
git branch -a 
#远程
git branch -r
#本地
git branch
#切换分支
git checkout

#push指定远程分支
git push origin $本地分支:$远程分支
git push origin master:test

#pull指定远程分支到当前branch
git push origin $远程分支
git push origin test
```

## git clean

```
git clean -n

是一次clean的演习, 告诉你哪些文件会被删除. 记住他不会真正的删除文件, 只是一个提醒

git clean -f

删除当前目录下所有没有track过的文件. 他不会删除.gitignore文件里面指定的文件夹和文件, 不管这些文件有没有被track过

```



## git rebase 和 git merge

### 合并多个commit为一个commit

当我们在本地仓库中提交了多次，在我们把本地提交push到公共仓库中之前，为了让提交记录更简洁明了，我们希望把如下分支B、C、D三个提交记录合并为一个完整的提交，然后再push到公共仓库。

![img](https:////upload-images.jianshu.io/upload_images/2147642-42195cacced56729.png?imageMogr2/auto-orient/strip|imageView2/2/w/417/format/webp)

现在我们在测试分支上添加了四次提交，我们的目标是把最后三个提交合并为一个提交：

这里我们使用命令:

```css
git rebase -i  [startpoint]  [endpoint]
```

其中`-i`的意思是`--interactive`，即弹出交互式的界面让用户编辑完成合并操作，`[startpoint]`  `[endpoint]`则指定了一个编辑区间，如果不指定`[endpoint]`，则该区间的终点默认是当前分支`HEAD`所指向的`commit`(注：该区间指定的是一个前开后闭的区间)。
 在查看到了log日志后，我们运行以下命令：

```undefined
git rebase -i 36224db
```

或:

```undefined
git rebase -i HEAD~3 
```

然后我们会看到如下界面:

```
pick 390b2ae test CICD
pick 9af9ef1 test CICD
pick abc95d6 test CICD
pick 7e0f538 test CICD

# Rebase f086b26..7e0f538 onto f086b26 (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, 编辑提交信息
# e, edit <commit> = use commit, 但停止修改
# s, squash <commit> = use commit, 压缩融合到之前的提交中
# f, fixup <commit> = like "squash", 但丢弃这个提交的日志混乱年龄
# x, exec <command> = 使用 shell 运行命令（行的其余部分）
# b, break = stop here (稍后使用 'git rebase --contin 继续 rebase
# d, drop <commit> = 删除提交
# l, label <label> = 用名称标记当前 HEAD
```

![img](https:////upload-images.jianshu.io/upload_images/2147642-03d48aa767efb307.png?imageMogr2/auto-orient/strip|imageView2/2/w/647/format/webp)
 上面未被注释的部分列出的是我们本次rebase操作包含的所有提交，下面注释部分是git为我们提供的命令说明。每一个commit id 前面的`pick`表示指令类型，git 为我们提供了以下几个命令:

> - pick：保留该commit（缩写:p）
> - reword：保留该commit，但我需要修改该commit的注释（缩写:r）
> - edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
> - squash：将该commit和前一个commit合并（缩写:s）
> - fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
> - exec：执行shell命令（缩写:x）
> - drop：我要丢弃该commit（缩写:d）

根据我们的需求，我们将commit内容编辑如下:

![img](https:////upload-images.jianshu.io/upload_images/2147642-a651234e62ed20a5.png?imageMogr2/auto-orient/strip|imageView2/2/w/536/format/webp)

然后是注释修改界面:



![img](https:////upload-images.jianshu.io/upload_images/2147642-44bbd784dcadfb31.png?imageMogr2/auto-orient/strip|imageView2/2/w/801/format/webp)

编辑完保存即可完成commit的合并了：

![img](https:////upload-images.jianshu.io/upload_images/2147642-334e0a5c47a24f87.png?imageMogr2/auto-orient/strip|imageView2/2/w/448/format/webp)

### 2.将某一段commit粘贴到另一个分支上

当我们项目中存在多个分支，有时候我们需要将某一个分支中的一段提交同时应用到其他分支中，就像下图：

![img](https:////upload-images.jianshu.io/upload_images/2147642-0de010746cb78401.png?imageMogr2/auto-orient/strip|imageView2/2/w/808/format/webp)


 我们希望将develop分支中的C~E部分复制到master分支中，这时我们就可以通过rebase命令来实现（如果只是复制某一两个提交到其他分支，建议使用更简单的命令:`git cherry-pick`）。
 在实际模拟中，我们创建了master和develop两个分支:
**master分支:**

![img](https:////upload-images.jianshu.io/upload_images/2147642-c41f60d26b00cdfc.png?imageMogr2/auto-orient/strip|imageView2/2/w/443/format/webp)

**develop分支:**

![img](https:////upload-images.jianshu.io/upload_images/2147642-8519a024c88129c5.png?imageMogr2/auto-orient/strip|imageView2/2/w/455/format/webp)

我们使用命令的形式为:

```css
    git rebase   [startpoint]   [endpoint]  --onto  [branchName]
```

其中，`[startpoint]`  `[endpoint]`仍然和上一个命令一样指定了一个编辑区间(前开后闭)，`--onto`的意思是要将该指定的提交复制到哪个分支上。
 所以，在找到C(90bc0045b)和E(5de0da9f2)的提交id后，我们运行以下命令：

```undefined
    git  rebase   90bc0045b^   5de0da9f2   --onto master
```

注:因为`[startpoint]`  `[endpoint]`指定的是一个前开后闭的区间，为了让这个区间包含C提交，我们将区间起始点向后退了一步。
 运行完成后查看当前分支的日志:

![img](https:////upload-images.jianshu.io/upload_images/2147642-de397671caac1966.png?imageMogr2/auto-orient/strip|imageView2/2/w/488/format/webp)

可以看到，C~E部分的提交内容已经复制到了G的后面了，大功告成？NO！我们看一下当前分支的状态:

![img](https:////upload-images.jianshu.io/upload_images/2147642-cfd21fdb1e4038bc.png?imageMogr2/auto-orient/strip|imageView2/2/w/439/format/webp)

当前HEAD处于游离状态，实际上，此时所有分支的状态应该是这样:

![img](https:////upload-images.jianshu.io/upload_images/2147642-a3bbfea6d760f64a.png?imageMogr2/auto-orient/strip|imageView2/2/w/755/format/webp)


 所以，虽然此时HEAD所指向的内容正是我们所需要的，但是master分支是没有任何变化的，`git`只是将C~E部分的提交内容复制一份粘贴到了master所指向的提交后面，我们需要做的就是将master所指向的提交id设置为当前HEAD所指向的提交id就可以了，即:

```undefined
      git checkout master
      git reset --hard  0c72e64
```

![img](https:////upload-images.jianshu.io/upload_images/2147642-003361cb0305c094.png?imageMogr2/auto-orient/strip|imageView2/2/w/689/format/webp)

此时我们才大功告成！





# 

![image-20220310130226770](http://myimg.go2flare.xyz/img/image-20220310130226770.png)

```
git pull origin test --allow-unrelated-histories
```

# PR模式

```
git提交流程记录：
个人创建分支：本地分支feature_liuchj、远程feature_liuchj

项目主干分支：本地feature分支、远程feature分支

1.首先将本地自己编写的代码add到本地暂存区【git add .     {注意有一个点符号，提交本地最近编写所有代码}】，然后commit到本地feature_liuchj分支.【git commit  -m "注释内容"】。

2.切换本地分支到本地feature分支【git checkout feature】

3.从远程feature分支拉取到本地feature分支【git pull origin feature】

4.切换本地分支到feature_liuchj【git checkout feature_liuchj】

5.Merge一下本地feature分支到本地feature_liuchj分支【git merge feature】

6.检查代码是否冲突，如果是代码有冲突则继续解决冲突后,重新提交到本地feature_liuchj 【自己看代码报错与否】

7.推送本地分支feature_liuchj的代码到远程分支feature_liuchj【git push remote feature_liuchj:feature_liuchj】

$ git push <远程主机名> <本地分支名>:<远程分支名>
 
git push origin feature_liuchj:feature_liuchj
 
#origin 为设置的远程仓库别名,feature_liuchj为本地分支名，feature_liuchj为远程分支名
8.在gitlab上面发送MergeRequest请求，将远程分支feature_liuchj请求合并到远程feature分支

9.由项目管理人员或者组长合并你的请求代码即可
```

# 忽略文件

## 全局忽略

```
1.在c盘->用户->正在登录的用户文件夹下 打开gitbash。

2.输入touch .gitignore_global，创建一个.gitignore_global文件。

3.用记事本打开或者vim编辑.gitignore_global，添加不需要上传的文件或者文件夹，文件夹后面要加/ 。

4.在gitbash执行 git config --global core.excludesfile ~/.gitignore_global，完成全局配置忽略上传。
```

ref:https://www.jianshu.com/p/62a412b7a6e8 

首先为什么要设置忽略文件呢？在进行协作开发代码管理的过程中，常常会遇到某些临时文件、配置文件、或者生成文件等，这些文件由于不同的开发端会不一样，如果使用git add . 将所有文件纳入git库中，那么会出现频繁的改动和push，这样会引起开发上的不便。

   Git可以很方便的帮助我们解决这个问题，那就是建立项目文件过滤规则。

## git忽略文件有三种: 

#### *1、全局范围内有效的忽略文件*

就是"版本库根目录/.git/info/exclude",全局范围内的所有忽略规则都以行为单位写在这个文件中;

#### 2、局部范围内有效的忽略文件

就是.gitignore,这个忽略文件只对某一级目录下的文件的忽略有效;

如果某一个目录下有需要被忽略的文件,那么就可以在该目录下手工地创建忽略文件.gitignore,

并在这个忽略文件中写上忽略规则,以行为单位,一条规则占据一行;

比较特殊的情况就是在版本库的根目录下创建一个忽略文件.gitignore,这时,

这个.gitignore忽略文件就对版本库根目录下的文件有效,等价于全局范围内的忽略文件.git/info/exclude;

#### 3、手工指定一个忽略文件, 

该忽略文件中的规则和语法与前两种是一致的,随便哪一级目录都可以,只要加上对应的路径即可;

手工指定忽略文件的命令是:

git config --global core.excludesfile /path/to/.gitignore

然后手工地在对应位置创建忽略文件.gitignore,并在该文件中写入忽略规则即可;

------

   **由于每一种级别的混略文件都是一致的所以今天主要介绍的是第二种.gitignore**

**1、配置语法：
**

以斜杠“/”开头表示目录；

以星号“*”通配多个字符；

以问号“?”通配单个字符

以方括号“[]”包含单个字符的匹配列表；

以叹号“!”表示不忽略(跟踪)匹配到的文件或目录；

**2、示例：**

（1）规则：fd1/*

说明：忽略目录 fd1 下的全部内容；注意，不管是根目录下的 /fd1/ 目录，还是某个子目录 /child/fd1/ 目录，都会被忽略；

（2）规则：/fd1/*

说明：忽略根目录下的 /fd1/ 目录的全部内容；

（3）规则：

/*

!.gitignore

!/fw/bin/

!/fw/sf/

说明：忽略全部内容，但是不忽略 .gitignore 文件、根目录下的 /fw/bin/ 和 /fw/sf/ 目录；

------

我们用coding管理代码，在创建项目的时候可以创建.gitignore文件

![img](https:////upload-images.jianshu.io/upload_images/1644584-45af8e057ae46079.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

我们来看生成的.gitignore文件

![img](https:////upload-images.jianshu.io/upload_images/1644584-baf343ff6aa980d3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

如果我们的项目中用到了cocopods的话很显然这些忽略文件是不够的，我们需要忽略podfile.lock文件



![img](https:////upload-images.jianshu.io/upload_images/1644584-b591c8dd36843aac.png?imageMogr2/auto-orient/strip|imageView2/2/w/861/format/webp)

------

#### **如果我们需要在当前工作目录添加文件忽略**

对于每一级工作目录，创建一个.gitignore文件，向该文件中添加要忽略的文件或目录。

但在创建并编辑这个文件之前，一定要保证要忽略的文件没有添加到git索引中。

使用命令git rm --cached filename将要忽略的文件从索引中删除。

# 常见问题

## 证书警告

```
warning: ----------------- SECURITY WARNING ----------------
warning: | TLS certificate verification has been disabled! |
warning: ---------------------------------------------------
warning: HTTPS connections may not be secure. See https://aka.ms/gcmcore-tlsverify for more information.
```

```
git config --global http.sslVerify true
```

## .gitignore push依然忽略不了文件

Git忽略规则(.gitignore配置）不生效原因和解决
参考文章：https://www.cnblogs.com/kevingrace/p/5690241.html

第一种方法:

.gitignore中已经标明忽略的文件目录下的文件，git push的时候还会出现在push的目录中，或者用git status查看状态，想要忽略的文件还是显示被追踪状态。
原因是因为在git忽略目录中，新建的文件在git中会有缓存，如果某些文件已经被纳入了版本管理中，就算是在.gitignore中已经声明了忽略路径也是不起作用的，
这时候我们就应该先把本地缓存删除，然后再进行git的提交，这样就不会出现忽略的文件了。

解决方法: git清除本地缓存（改变成未track状态），然后再提交:
[root@kevin ~]# git rm -r --cached .
[root@kevin ~]# git add .
[root@kevin ~]# git commit -m 'update .gitignore'
[root@kevin ~]# git push -u origin master

需要特别注意的是：
1）.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。
2）想要.gitignore起作用，必须要在这些文件不在暂存区中才可以，.gitignore文件只是忽略没有被staged(cached)文件，
   对于已经被staged文件，加入ignore文件时一定要先从staged移除，才可以忽略。

第二种方法:（推荐）
在每个clone下来的仓库中手动设置不要检查特定文件的更改情况。
[root@kevin ~]# git update-index --assume-unchanged PATH                  //在PATH处输入要忽略的文件

在使用.gitignore文件后如何删除远程仓库中以前上传的此类文件而保留本地文件
在使用git和github的时候，之前没有写.gitignore文件，就上传了一些没有必要的文件，在添加了.gitignore文件后，就想删除远程仓库中的文件却想保存本地的文件。这时候不可以直接使用"git rm directory"，这样会删除本地仓库的文件。可以使用"git rm -r –cached directory"来删除缓冲，然后进行"commit"和"push"，这样会发现远程仓库中的不必要文件就被删除了，以后可以直接使用"git add -A"来添加修改的内容，上传的文件就会受到.gitignore文件的内容约束。

额外说明：git库所在的文件夹中的文件大致有4种状态

Untracked:

未跟踪, 此文件在文件夹中, 但并没有加入到git库, 不参与版本控制. 通过git add 状态变为Staged.

Unmodify:
文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改,
而变为Modified. 如果使用git rm移出版本库, 则成为Untracked文件

Modified:
文件已修改, 仅仅是修改, 并没有进行其他的操作. 这个文件也有两个去处, 通过git add可进入暂存staged状态,
使用git checkout 则丢弃修改过, 返回到unmodify状态, 这个git checkout即从库中取出文件, 覆盖当前修改

Staged:
暂存状态. 执行git commit则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为Unmodify状态.
执行git reset HEAD filename取消暂存, 文件状态为Modified

Git 状态 untracked 和 not staged的区别
1）untrack     表示是新文件，没有被add过，是为跟踪的意思。
2）not staged  表示add过的文件，即跟踪文件，再次修改没有add，就是没有暂存的意思

## 无法完全push

```
Total 1119 (delta 545), reused 0 (delta 0), pack-reused 0
fatal: the remote end hung up unexpectedly
```

```
git config --global http.postBuffer 524288000
```

这个重新建个分支pull再merge后，重新push也行

## push rejected

```
To https://gitlab.com/Go2Flare/cicd_repo.git
 ! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'https://gitlab.com/Go2Flare/cicd_repo.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

原因：远程新建项目的时候有readme文件，跟本地仓库的有冲突

方法1：（不推荐）

强制覆盖本地仓库

```
git push -f 仓库名 分支
```

方法2：

将远程先合并pull，再push即可

```
git pull origin master --allow-unrelated-histories (该选项可以合并两个独立启动仓库的历史)
git push -u origin master
```

## [删除远程仓库记录](https://dalewushuang.blog.csdn.net/article/details/102489362?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_paycolumn_v3&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_paycolumn_v3)

ref:https://www.cnblogs.com/zhaotianff/p/13860307.html

查了下资料，终于找到简单粗暴的方式来删除提交记录。方法如下

```
1 git reset --soft HEAD~i
```

**i代表要恢复到多少次提交前的状态，如指定i = 2，则恢复到最近两次提交前的版本。--soft代表只删除服务器记录，不删除本地。**

再执行

```
1 git push origin master --force
```

**master代表当前分支**

这样操作完成后，服务器最近的两次提交记录已经看不到了。

**此时，我们再把本地的文件提交一次就行了。**

 
