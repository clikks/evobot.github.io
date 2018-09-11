---
title: GIT分支与标签管理
author: Evobot
date: 2018-09-09 23:53:03
categories: 代码管理平台
tags:
  - GIT
image:
---

1. 分支管理
2. 远程分支管理
3. 标签管理
4. git别名

<!--more-->

---



# 本地分支管理

## 创建分支

- 分支在git中是非常常用的功能，在仓库中使用`git branch`查看分支：

  ```bash
  $ git branch
  * master
  # '*'表示当前分支
  ```

- `git branch [branch_name]`命令用来创建一个名为“branch_name"的新分支：

  ```bash
  [root@localhost gitroot]# git branch test
  [root@localhost gitroot]# git branch
  * master
    test
  
  ```

- 命令`git checkout [branch_name]`切换到指定分支：

  ```bash
  [root@vm1 gitroot]# git checkout test
  切换到分支 'test'
  [root@vm1 gitroot]# git branch
    master
  * test
  
  ```

- 新创建的分支内，文件内容与原master分支是相同的，但在新的分支下创建文件并commit后，文件就只会存在于新分支：

  ```bash
  [root@vm1 gitroot]# git branch
    master
  * test
  [root@vm1 gitroot]# ls
  2.txt  README.md
  [root@vm1 gitroot]# touch test_file.txt
  [root@vm1 gitroot]# ls
  2.txt  README.md  test_file.txt
  
  [root@vm1 gitroot]# git add test_file.txt 
  [root@vm1 gitroot]# git commit -m 'add new file test_file.txt'
  [test 9243970] add new file test_file.txt
   1 file changed, 0 insertions(+), 0 deletions(-)
   create mode 100644 test_file.txt
  
  [root@vm1 gitroot]# git checkout master
  切换到分支 'master'
  [root@vm1 gitroot]# ls
  2.txt  README.md
  
  ```

## 合并分支

- 实际使用中，master分支作为我们的主分支，另外创建新分支来进行下一步开发，当开发完后，可以将开发分支进行合并到主分支或与其他开发分支合并；

- 合并分支之前，要先切换到需要合并入的目标分支，例如将test分支合并到master分支，需要先切换到master分支；

- 然后使用`git merge [branch]`命令将指定的分支合并到当前分支：

  ```bash
  [root@vm1 gitroot]# git branch
  * master
    test
  [root@vm1 gitroot]# git merge test
  更新 3a9ad27..9243970
  Fast-forward
   test_file.txt | 0
   1 file changed, 0 insertions(+), 0 deletions(-)
   create mode 100644 test_file.txt
  [root@vm1 gitroot]# ls
  2.txt  README.md  test_file.txt
  ```

- 在实际开发中，可能会存在多个分支的用户都对一个文件进行了修改和提交，这样的两个分支在合并时，就会产生冲突，产生冲突时，必须先解决冲突，才能继续进行合并：

  ```bash
  [root@vm1 gitroot]# echo 'andsio' > test_file.txt 
  
  # git commit -am等同于git add . && git commit -m，但使用这条命令的文件必须是已经被版本库所跟踪的文件。
  [root@vm1 gitroot]# git commit -am 'ch test_file'
  [master 67a8ba6] ch test_file
   1 file changed, 1 insertion(+)
  [root@vm1 gitroot]# git status
  # 位于分支 master
  # 您的分支领先 'origin/master' 共 2 个提交。
  #   （使用 "git push" 来发布您的本地提交）
  #
  无文件要提交，干净的工作区
  
  [root@vm1 gitroot]# git checkout test
  切换到分支 'test'
  [root@vm1 gitroot]# echo 'asfasd' > test_file.txt 
  [root@vm1 gitroot]# git commit -am 'ch test_file'
  [test 67a729f] ch test_file
   1 file changed, 1 insertion(+)
   
  [root@vm1 gitroot]# git checkout master
  切换到分支 'master'
  您的分支领先 'origin/master' 共 2 个提交。
    （使用 "git push" 来发布您的本地提交）
  [root@vm1 gitroot]# git merge test
  自动合并 test_file.txt
  冲突（内容）：合并冲突于 test_file.txt
  自动合并失败，修正冲突然后提交修正的结果。
  
  ```

- 对于产生冲突的文件，git会在文件中进行标记：

  ```bash
  [root@vm1 gitroot]# cat test_file.txt 
  <<<<<<< HEAD
  andsio
  =======	# ===上面是当前分支commit的内容，下面则是test分支提交的内容
  asfasd
  >>>>>>> test
  
  ```

- 一般解决冲突，是在当前分支下，编辑冲突的文件，将其改为与被合并分支内容一致，然后再重新合并；但如果当前分支的更改内容是我们所需要的，也可将文件改为与当前分支相同，提交后切换到被合并分支，然后将最新的分支合并到旧分支即可，**也就是说，merge命令后面的分支必须是最新的分支**：

  ```bash
  [root@vm1 gitroot]# sed -i '1,3d;5d' test_file.txt 
  [root@vm1 gitroot]# git add test_file.txt 
  [root@vm1 gitroot]# git commit -m 'fix test_file'
  [master 17806f9] fix test_file
  
  # commit之后，git会自动完成merge
  [root@vm1 gitroot]# git status
  # 位于分支 master
  # 您的分支领先 'origin/master' 共 4 个提交。
  #   （使用 "git push" 来发布您的本地提交）
  #
  无文件要提交，干净的工作区
  [root@vm1 gitroot]# git reflog
  17806f9 HEAD@{0}: commit (merge): fix test_file
  67a8ba6 HEAD@{1}: checkout: moving from test to master
  67a729f HEAD@{2}: commit: ch test_file
  9243970 HEAD@{3}: checkout: moving from master to test
  67a8ba6 HEAD@{4}: commit: ch test_file
  9243970 HEAD@{5}: merge test: Fast-forward
  3a9ad27 HEAD@{6}: checkout: moving from test to master
  ...
  
  ```

  ```bash
  //保留master的内容并合并
  [root@vm1 gitroot]# echo 'adsas' >> test_file.txt 
  [root@vm1 gitroot]# git commit -am 'ch test_file'
  [master 15f2d97] ch test_file
   1 file changed, 1 insertion(+)
  
  [root@vm1 gitroot]# git checkout test
  切换到分支 'test'
  [root@vm1 gitroot]# echo 'asda' >> test_file.txt 
  [root@vm1 gitroot]# git commit -am 'ch test_file'
  [test fda9cfc] ch test_file
   1 file changed, 1 insertion(+)
   
  [root@vm1 gitroot]# git checkout master
  切换到分支 'master'
  您的分支领先 'origin/master' 共 5 个提交。
    （使用 "git push" 来发布您的本地提交）
  [root@vm1 gitroot]# git merge test
  自动合并 test_file.txt
  冲突（内容）：合并冲突于 test_file.txt
  自动合并失败，修正冲突然后提交修正的结果。
  [root@vm1 gitroot]# cat test_file.txt 
  asfasd
  <<<<<<< HEAD
  adsas
  =======
  asda
  >>>>>>> test
  
  [root@vm1 gitroot]# sed -i '2d;4,6d' test_file.txt 
  [root@vm1 gitroot]# git commit -am 'fix test_file'
  # commit后master已经是最新的分支
  
  [root@vm1 gitroot]# git checkout test
  切换到分支 'test'
  [root@vm1 gitroot]# git merge master
  更新 fda9cfc..3c91aed
  Fast-forward
   test_file.txt | 2 +-
   1 file changed, 1 insertion(+), 1 deletion(-)
  [root@vm1 gitroot]# git reflog
  3c91aed HEAD@{0}: merge master: Fast-forward
  fda9cfc HEAD@{1}: checkout: moving from master to test
  3c91aed HEAD@{2}: commit (merge): fix test_file
  15f2d97 HEAD@{3}: checkout: moving from test to master
  ...
  ```

## 删除分支

- `git branch -d [branch]`可以删除分支，删除分支不能删除当前分支，要先切换到其他分支，并且如果分支没有合并，在删除前会提示：

  ```bash
  [root@vm1 gitroot]# git branch
    master
  * test
  [root@vm1 gitroot]# git branch -d test
  error: 无法删除您当前所在的分支 'test'。
  
  [root@vm1 gitroot]# echo 'asda' >> test_file.txt 
  [root@vm1 gitroot]# git commit -am 'ch test_file'
  [test 2c1b7bf] ch test_file
   1 file changed, 1 insertion(+)
  
  [root@vm1 gitroot]# git checkout master
  切换到分支 'master'
  您的分支领先 'origin/master' 共 7 个提交。
    （使用 "git push" 来发布您的本地提交）
  
  [root@vm1 gitroot]# git branch -d test
  error: 分支 'test' 没有完全合并。
  如果您确认要删除它，执行 'git branch -D test'。
  
  ```

- 如果不打算合并分支并且要将分支删除，使用`git branch -D [branch]`强制删除分支：

  ```bash
  [root@vm1 gitroot]# git branch -D test
  已删除分支 test（曾为 2c1b7bf）。
  
  ```

# 分支使用原则

- 对于分支的应用，一般建议以以下原则进行使用：

1. master分支是非常重要的，线上发布代码一般使用这个分支，平时进行开发作业不要在master分支上进行；

2. 创建一个dev分支，专门用作开发，只有当发布到线上之前，才将dev分支合并到master分支；

3. 开发人员应该在dev的基础上再创建分支作为自己的个人分支，开发人员在各自的个人分支里进行代码开发，然后合并到dev分支，如下图：

   ![ikmCNQ.png](https://s1.ax1x.com/2018/09/11/ikmCNQ.png)


# 远程分支管理

- 本地克隆远程仓库时，只能克隆master分支到本地；

- 命令`git ls-remote origin`命令可以查看远程仓库所有的分支：

  ```bash
  [root@vm1 gitroot]# git ls-remote origin
  3a9ad278e1ea1119c7e65fd2fe76b0a1ad71e034	HEAD
  3a9ad278e1ea1119c7e65fd2fe76b0a1ad71e034	refs/heads/master
  
  ```

- 我们在github上为远程仓库创建一个新的dev分支：

  [![远程创建分支J.md.png](https://s1.ax1x.com/2018/09/11/ikmM4J.md.png)](https://imgchr.com/i/ikmM4J)

  - 创建完成后，在本地克隆的仓库内可以看到远程仓库的dev分支：

  ```bash
  [root@vm1 gitroot]# git ls-remote origin
  3a9ad278e1ea1119c7e65fd2fe76b0a1ad71e034	HEAD
  3a9ad278e1ea1119c7e65fd2fe76b0a1ad71e034	refs/heads/dev
  3a9ad278e1ea1119c7e65fd2fe76b0a1ad71e034	refs/heads/master
  
  ```

- 因为本地克隆时只能克隆下master分支，如果要将远程dev分支与本地关联，则执行`git checkout -b [branch] origin/[branch]`命令：

  ```bash
  [root@vm1 evobot]# git checkout -b dev origin/dev
  分支 dev 设置为跟踪来自 origin 的远程分支 dev。
  切换到一个新分支 'dev'
  [root@vm1 evobot]# git branch
  * dev
    master
  ```

- 切换到新的dev分支，创建文件，并提交到远程分支，git会自动将本地dev分支的commit内容提交到远程对应的dev分支：

  ```bash
  [root@vm1 evobot]# echo 'asnoasfn' > a.txt
  [root@vm1 evobot]# git add a.txt 
  [root@vm1 evobot]# git commit -m 'create a.txt'
  [dev 450fd00] create a.txt
   1 file changed, 1 insertion(+)
   create mode 100644 a.txt
   
  [root@vm1 evobot]# git push
  Counting objects: 4, done.
  Compressing objects: 100% (2/2), done.
  Writing objects: 100% (3/3), 298 bytes | 0 bytes/s, done.
  Total 3 (delta 0), reused 0 (delta 0)
  To git@github.com:clikks/evobot.git
     3a9ad27..450fd00  dev -> dev
  
  ```

- 当我们与远程对应的多个本地分支都存在更新时，在Git 1.x中，执行`git push`会将所有分支的更改提交的远程对应分支中，称为`marching`方式；在Git 2.x中，则只会push当前分支的更改到远程分支，称之为`simple`方式；

- 在我们首次使用git时，git会提示下面两条命令用来设置Git全局push方式：

  ```bash
  git config --global push.default matching
  git config --global push.default simple
  ```

- 在执行`git push`命令时，如果我们只推送指定分支，则可以执行`git push origin [branch]`：

  ```bash
  [root@vm1 evobot]# git push origin dev
  Counting objects: 5, done.
  Compressing objects: 100% (2/2), done.
  Writing objects: 100% (3/3), 254 bytes | 0 bytes/s, done.
  Total 3 (delta 1), reused 0 (delta 0)
  remote: Resolving deltas: 100% (1/1), completed with 1 local object.
  To git@github.com:clikks/evobot.git
     450fd00..5d982dd  dev -> dev
  ```

- 当本地存在远程仓库不存在的分支时，执行`git push`，Git会提示以下信息：

  ```bash
  [root@vm1 evobot]# git branch dev2
  [root@vm1 evobot]# git branch
  * dev
    dev2
    master
  [root@vm1 evobot]# git checkout dev2
  切换到分支 'dev2'
  [root@vm1 evobot]# echo 'asdas' >> a.txt 
  [root@vm1 evobot]# git add .
  [root@vm1 evobot]# git commit -m 'ch a.txt'
  [dev2 af42b42] ch a.txt
   1 file changed, 1 insertion(+)
  
  //simple方式的push
  [root@vm1 evobot]# git push
  fatal: 当前分支 dev2 没有对应的上游分支。
  为推送当前分支并建立与远程上游的跟踪，使用
  
      git push --set-upstream origin dev2
  
  //matching方式的push
  [root@vm1 evobot]# git push
  Everything up-to-date
  ```

- 对于上面的情况，我们推送时，需要执行`git push origin [branch]`(matching)方式：

  ```bash
  [root@vm1 evobot]# git push origin dev2
  Counting objects: 5, done.
  Compressing objects: 100% (2/2), done.
  Writing objects: 100% (3/3), 256 bytes | 0 bytes/s, done.
  Total 3 (delta 1), reused 0 (delta 0)
  remote: Resolving deltas: 100% (1/1), completed with 1 local object.
  remote: 
  remote: Create a pull request for 'dev2' on GitHub by visiting:
  remote:      https://github.com/clikks/evobot/pull/new/dev2
  remote: 
  To git@github.com:clikks/evobot.git
   * [new branch]      dev2 -> dev2
  
  ```

- 而simple方式推送新的分支，则需要执行`git push --set-upstream origin [branch]`，其中`--set-upstream`用于将本地分支与远程分支进行关联：

  ```bash
  [root@vm1 evobot]# git push --set-upstream origin dev4
  Counting objects: 5, done.
  Compressing objects: 100% (2/2), done.
  Writing objects: 100% (3/3), 253 bytes | 0 bytes/s, done.
  Total 3 (delta 1), reused 0 (delta 0)
  remote: Resolving deltas: 100% (1/1), completed with 1 local object.
  remote: 
  remote: Create a pull request for 'dev4' on GitHub by visiting:
  remote:      https://github.com/clikks/evobot/pull/new/dev4
  remote: 
  To git@github.com:clikks/evobot.git
   * [new branch]      dev4 -> dev4
  分支 dev4 设置为跟踪来自 origin 的远程分支 dev4。
  
  ```

# 标签管理

- 标签类似于快照功能，可以给版本库打一个标签，记录某个时刻库的状态，也可以随时恢复到该状态，常用来对代码发布版本打标签，一般打标签是在master分支上进行操作；

- `git tag [lable]`命令用来给当前分支的版本库状态打标签，直接执行`git tag`可以查看标签：

  ```bash
  
  ```

- `git show [label]`可以查看指定标签的信息：

  ```bash
  [root@vm1 evobot]# git show v1.0
  commit 3a9ad278e1ea1119c7e65fd2fe76b0a1ad71e034
  Author: evobot <admin@evobot.cn>
  Date:   Sun Sep 9 23:37:24 2018 +0800
  
      add 2.txt
  
  diff --git a/2.txt b/2.txt
  new file mode 100644
  index 0000000..ae59dd3
  --- /dev/null
  +++ b/2.txt
  @@ -0,0 +1 @@
  +this is 2.txt
  
  ```

  - 这里可以看到，标签是针对commit来进行标记的

- 因为tag是针对commit进行标记的，所以我们也可以针对指定的commit历史进行打标签，使用`git tag [label] [ID]`命令：

  ```bash
  [root@vm1 evobot]# git log --pretty=oneline
  3a9ad278e1ea1119c7e65fd2fe76b0a1ad71e034 add 2.txt
  bf4c28c79e61426b287af1143e569734626f73e4 add README.md
  7d13f758af781c9c0b008caf861e1f68c0f06504 rm 1.txt
  8e0da3ff80b115d57bbd06c6fa710583ed5fa9b1 change 1.txt
  13a572e092ee57f56b52ab7a5d4865baf4336125 add new file 1.txt
  [root@vm1 evobot]# git tag v0.8 bf4c28c7
  [root@vm1 evobot]# git tag
  v0.8
  v1.0
  [root@vm1 evobot]# git show v0.8
  commit bf4c28c79e61426b287af1143e569734626f73e4
  Author: evobot <admin@evobot.cn>
  Date:   Sun Sep 9 23:31:14 2018 +0800
  
      add README.md
  
  diff --git a/README.md b/README.md
  new file mode 100644
  index 0000000..78585d2
  --- /dev/null
  +++ b/README.md
  @@ -0,0 +1 @@
  +# EVOBOT repository
  
  ```

- `git log --pretty=oneline --abbrev-commit`命令中的`--abbrev-commit`可以将commit的ID进行简单显示：

  ```bash
  [root@vm1 evobot]# git log --pretty=oneline --abbrev-commit
  3a9ad27 add 2.txt
  bf4c28c add README.md
  7d13f75 rm 1.txt
  8e0da3f change 1.txt
  13a572e add new file 1.txt
  
  ```

- 对于标签，我们还可以在创建标签时为标签增加相应的描述，命令为`git tag -a [label] -m [description] [ID]`：

  ```bash
  [root@vm1 evobot]# git tag -a v0.1 -m 'tag just v0.1 and so on' 13a572e
  [root@vm1 evobot]# git tag
  v0.1
  v0.8
  v1.0
  [root@vm1 evobot]# git show v0.1
  tag v0.1
  Tagger: evobot <admin@evobot.cn>
  Date:   Tue Sep 11 23:21:35 2018 +0800
  
  tag just v0.1 and so on
  
  commit 13a572e092ee57f56b52ab7a5d4865baf4336125
  Author: root <root@localhost.localdomain>
  Date:   Sun Sep 9 22:19:31 2018 +0800
  
      add new file 1.txt
  
  diff --git a/1.txt b/1.txt
  new file mode 100644
  index 0000000..b149eee
  --- /dev/null
  +++ b/1.txt
  @@ -0,0 +1,4 @@
  +123
  +aaa
  +456
  +bbb
  
  ```

- 删除指定的标签，使用`git tag -d [label]`：

  ```bash
  [root@vm1 evobot]# git tag -d v0.1
  已删除 tag 'v0.1'（曾为 14d0180）
  ```

- 在github上的branch选择按钮中，不仅可以选择分支，也可以对标签进行选择；

- 将本地的标签推送到远程仓库，使用命令为`git push origin [label]`：

  ```bash
  [root@vm1 evobot]# git push origin v1.0
  Total 0 (delta 0), reused 0 (delta 0)
  To git@github.com:clikks/evobot.git
   * [new tag]         v1.0 -> v1.0
  
  ```

- 推送所有的标签到远程仓库，使用`git push --tag origin`命令：

  ```bash
  [root@vm1 evobot]# git push --tag origin
  Total 0 (delta 0), reused 0 (delta 0)
  To git@github.com:clikks/evobot.git
   * [new tag]         v1.1 -> v1.1
   * [new tag]         v1.2 -> v1.2
  
  ```

- 如果本地删除了一个标签，并且将远程对应的标签也进行删除，则使用命令`git push origin :refs/tags/[label]`：

  ```bash
  [root@vm1 evobot]# git tag -d v1.2
  已删除 tag 'v1.2'（曾为 3a9ad27）
  [root@vm1 evobot]# git push origin :refs/tags/v1.2
  To git@github.com:clikks/evobot.git
   - [deleted]         v1.2
  
  ```

# Git别名

- git同样支持命令别名，使用别名可以提高我们的效率；

- `git config --global alias.[cmd] [command]`命令是git设置命令别名的方式，cmd是我们设置的别名，command则是git的命令：

  ```bash
  [root@vm1 evobot]# git config --global alias.ci commit
  [root@vm1 evobot]# echo 'asda' >> 2.txt 
  (reverse-i-search)`': ^Ct branch
  [root@vm1 evobot]# git add .
  [root@vm1 evobot]# git ci -m 'ch 2.txt'
  [master 69a08c5] ch 2.txt
   1 file changed, 1 insertion(+)
  
  ```

  ```bash
  [root@vm1 evobot]# git config --global alias.br branch
  [root@vm1 evobot]# git br
    dev
  * master
  
  ```

- 查看git别名使用的命令，使用`git config --list | grep alias`：

  ```bash
  [root@vm1 evobot]# git config --list | grep alias
  alias.ci=commit
  alias.br=branch
  
  ```

  - git config的配置文件在家目录下，文件名为`.gitconfig`：

  ```bash
  [root@vm1 evobot]# cat /root/.gitconfig 
  [user]
  	name = evobot
  	email = admin@evobot.cn
  [push]
  	default = simple
  [alias]
  	ci = commit
  	br = branch
  
  ```

- 我们可以将一些特别的命令设置为别名方便我们使用，例如查看git log，可以设置下面的别名，实现执行`git lg`输出带颜色和格式的log：

  ```bash
  git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset-%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
  ```

  ```bash
  [root@vm1 evobot]# git lg
  * 69a08c5- (HEAD, master) ch 2.txt (9 分钟之前) <evobot>
  * 3a9ad27- (tag: v1.1, tag: v1.0, origin/master, origin/HEAD) add 2.txt (2 天之前) <evobot>
  * bf4c28c- (tag: v0.8) add README.md (2 天之前) <evobot>
  * 7d13f75- rm 1.txt (2 天之前) <evobot>
  * 8e0da3f- change 1.txt (2 天之前) <evobot>
  * 13a572e- add new file 1.txt (2 天之前) <root>
  
  ```

- 取消别名，使用`git config --global --unset alias.[cmd]`：

  ```bash
  [root@vm1 evobot]# git config --global --unset alias.br
  [root@vm1 evobot]# git br
  git：'br' 不是一个 git 命令。参见 'git --help'。
  
  您指的是这其中的某一个么？
  	branch
  	var
  
  ```

---