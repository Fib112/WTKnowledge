[TOC]

### git仓库创建

##### git仓库初始化

git init可以将当前目录变为git仓库

##### 提交文件

1. 使用git add <file name>将指定文件提交到暂存区
2. 使用git commit -m "..." 将暂存区文件提交至仓库,引号中填写提交说明

### git版本控制

##### 工作区状态命令

1. git status可以查看工作区和暂存区状态，比如哪些文件被修改了，暂存区有哪些文件要被提交到仓库
2. git diff可以查看文本具体修改的内容

##### 版本回退

1. git log可以查看提交日志 加上--pretty=oneline可以将内容显示到一行
2. 在git中HEAD表示当前版本，HEAD^则表示上一个版本，HEAD^^则表示上上个版本，当^较多时可以使用HEAD~x，x表示^的个数。然后使用git reset --hard HEAD^可以回退到上一个版本。
3. 此时用git log查看可以发现HEAD指针指向回退的版本，但最新那个版本不见了。若要找到这个版本可以使用git reflog命令查看历史命令。
4. 找到最新的文件的commit id后，使用git reset --hard commit id后可以回退到最新版本

##### 撤销修改

1. 使用git checkout -- <file name>可以把该文件在工作区的修改全部撤销。此时有两种情况
   1. 该文件自修改后还未被放到暂存区，此时撤销则回到和版本库一摸一样的状态
   2. 该文件添加到暂存区后做了修改，则此时会回到和暂存区一样的状态
2. 如果修改后添加到了暂存区，可以用git reset HEAD <file name>来将暂存区修改撤销重新回到工作区
3. 如果修改后将文件提交到了仓库中，则要使用git reset xxx等版本回退指令进行撤销

##### 删除文件

- 在工作区目录下，使用rm <file name>可以删除一个文件。这只是删除工作目录下的文件，此时有两个选择：

  1. 将该文件从版本库中删除

     使用两条指令：1. git rm <file name> 2. git commit -m "remove <filename>"

     此时文件从工作目录和仓库中删除

  2. 误删文件，需要恢复

     使用git checkout -- <file name>从暂存区或者版本库中恢复

### 远程仓库

1. 创建SSH Key。打开git bash

   ```shell
   $ ssh-keygen -t rsa -C "emailaddress"
   ```

   然后找到用户主目录下的.ssh，，里面有id_rsa和id_rsa.pub，其中id_rsa是私钥，不能泄露，id_rsa.pub是公钥

2. 登录GitHub，打开Account settings的SSH Keys点Add SSH Key，填上任意title，在key文本框粘贴id_rsa.pub的内容

3. 创建GitHub新仓库。

4. 在本地工作目录运行命令

   ```
   $ git remote add origin
   git@github.com:username/reponame.git //username为GitHub用户名，reponame为仓库名
   ```

5. 将本地库推送到GitHub

   ```
   $ git push -u origin master
   ```

   **SSH警告：第一次使用clone或push连接git时会得到警告，在输入前对照GitHub的RSA Key的指纹信息是否与SSH连接给出的一致后输入yes即可**

6. 本地提交

   ```
   $ git push origin master //可将本地master分支的最新修改推送至GitHub
   ```

7. 删除远程库

   ```
   $ git remote rm <name>
   //使用前可先运行以下代码查看远程库信息
   $ git remove -v
   ```

### 分支管理

##### 创建与合并分支

- 实际上HEAD指针并不是指向最新提交，而是指向当前分支，而master才是指向主分支的最新提交。当创建新分支时，会生成一个dev指针指向master所指的节点，而HEAD指针指向dev指针。而当做了一次新提交后，master指针不变，dev指针指向新节点。当合并时，只需要将master指针指向dev所指节点并将HEAD指针指向master。当删除分支时只需要删除dev指针即可。创建分支指令如下

	```
	$ git checkout -b dev //git checkout命令加上-b参数表示创建并切换，相当于以下两条命	令
	$ git branch dev
	$ git checkout dev
	```

- 然后用git branch命令查看当前分支：

	```
	$ git branch //git branch命令会列出所有分支，当前分支会加上星号
	* dev
    master
	```

- 合并分支

	```
	$ git merge dev
	```

- 删除分支

	```
	$ git branch -d dev
	```
	
- 新的切换命令

  切换分支使用命令$ git checkout <branch>，而前面说过撤销修改是$ git checkout -- <file name>，同一个命令有两种命令会造成疑惑，所以git提供了新的切换命令

  ```
  //创建并切换到新的dev分支，可以使用:
  $ git switch -c dev
  //直接切换到已有的master，可以使用:
  $ git switch master
  ```

##### 解决冲突

当master与分支提交存在冲突时就会无法合并，必须手动修改冲突

![合并冲突](https://www.liaoxuefeng.com/files/attachments/919023000423040/0)

使用git status可以查看冲突的文件

手动修改文件并add和commit后冲突会修复，合并成功

![image](https://www.liaoxuefeng.com/files/attachments/919023031831104/0)

使用git log --graph命令可以看到合并分支图

最后删除分支

```
$ git branch -d feature1
```

##### 分支管理策略

通常合并分支时，如果可能，git会使用Fast forward模式，但这在删除分支后会丢失分支信息。如果禁用Fast forward模式，git会在merge时生成新的commit，这样从分支历史就可以看出分支信息

- 首先创建dev分支

  ```
  $ git switch -c dev
  //修改readme文件，提交新的commit
  $ git add readme.txt
  $ git commit -m "add merge"
  //切换回master
  $ git switch master
  ```

- 准备合并master，--no--ff参数表示禁用Fast forward模式

  ```
  $ git merge --no--ff -m "merge with no--ff" dev
  //因为本次合并要创建一个新的commit，所以加上-m参数，把commit描述写进去
  ```

- 合并后，用git log查看分支历史：

  ```
  $ git log --graph --pretty=oneline --abbrev-commit
  ```

- 不使用Fast forward模式，merge后像这样：

  ![mergeimg](https://www.liaoxuefeng.com/files/attachments/919023225142304/0)

- 在实际开发中，应该按照以下原则进行分支管理：

  1. **master分支应该是稳定的**，也就是仅用来发布新版本，平时不能在上面干活

  2. 平时干活应该在dev分支上，也就是说dev是不稳定的，到某个时候需要发布新版本，再把dev分支合并到master上，在master分支发布新版本

  3. 每个人都在dev分支上干活，每个人都有自己分支，时不时往dev分支上合并就行了

     如图所示：

     ![groupwork](https://www.liaoxuefeng.com/files/attachments/919023260793600/0)
     
##### bug分支

  - 当接到一个修复代号为101的bug的任务时，会想创建一个分支来修复它，但当前在dev分支上的工作还没提交。而此时工作仅进行到一半无法提交，但此时又必须去修复bug。这时，可以用stash功能将工作现场“储藏”起来，等以后恢复现场后继续工作

    ```
    $ git stash
    ```

    此时用git status查看工作区是干净的，可以用来修复bug

  - 然后确定要在哪个分支修复bug，比如要在master分支修复bug，就从master创建临时分支

    ```
    $ git switch master
    ...
    $ git switch -c issue-101
    ...
    ```

    之后修复bug，提交

    ```
    $ git add readme.txt
    $ git commit -m "fix bug 101"
    [issue-101 4c805e2] fix bug 101
     1 file changed, 1 insertion(+), 1 deletion(-)
    ```

    修复完成后，切换到master分支完成合并

    ```
    $ git switch master
    ...
    $ git merge --no--ff -m "merged bug fix 101" issue-101
    ...
    ```

    最后切换回dev分支干活

  - 此时工作区是干净的，用git  stash list命令查看工作现场存在哪了

    ```
    $ git stash list
    ...
    ```

    此时恢复有两种办法：

    1. 使用git stash apply恢复，但恢复后stash内容不删除，需要用git stash drop来删除
    2. 使用git stash pop，在恢复的同时将stash内容删了

    此时再用git stash list查看就看不到任何stash内容了

  - 可以多次stash，恢复时先用git stash list查看，然后恢复指定stash，用命令：`$ git stash apply stash@{0}`

  - 在master分支修复完bug后，dev分支是早期从master分支分出来的，所以这个bug在dev分支也存在。同样的bug在dev分支上修复存在便捷方法：将在master分支上提交的修复bug的提交复制到dev分支，比如在上面的4c805e2 fix bug 101提交，而不是将整个master分支merge过来。<br>为了方便操作，git提供了cherry-pick命令，能复制一个特定的提交到当前分支：

    ```
    //当前在dev分支
    $ git cherry-pick 4c805e2
    [master 1d4b803] fix bug 101
     1 file changed, 1 insertion(+), 1 deletion(-)
    ```

    git自动给dev分支做了一次提交，注意这次提交commit是1d4b803，并不同于master的4c805e2。因为这两个commit只是改动相同，但是不同的两个commit。

  - 既然可以在master分支修复bug后在dev分支“重放”这个过程，那么同理也可以在dev分支修复bug，然后在master分支重放。但仍然需要使用git stash命令保存现场，才能从dev分支切换到master分支

##### Feature分支

- 当需要开发新功能时，最好新建一个feature分支，在上面开发，完成后，合并，最后删除该分支

  ```
  $ git switch -c feature-vulcan
  ...
  //开发完毕后
  $ git add vulcan.py
  $ git commit -m "add feature vulcan"
  ```

- 然后切换回dev准备合并

  ```
  $ git switch dev
  ```

  但此时接到命令，经费不足，取消新功能，所以只能删除feature分支

  ```
  $ git branch -d feature-vulcan
  error: xxxxxxxx
  ```

  销毁失败，git提示feature-vulcan分支没有被合并，如果删除会丢失修改。如果要强行修改需要使用-D参数。

  现在强行删除

  ```
  $ git branch -D feature-vulcan
  //删除成功
  ```

##### 多人协作

- 从远程仓库克隆时，实际上git自动把本地的master分支和远程的master分支对应起来了，并且远程仓库的默认名称是origin。

  要查看远程库的信息，用git remote：

  ```
  $ git remote
  origin
  //用git remote -v可以显示更详细的信息
  $ git remote -v
  ...
  ...
  //上面显示可以fetch和push的origin的地址
  ```

- 推送分支

  推送分支就是把该分支的所有本地提交推送到远程仓库，推送时要指定本地分支

  ```
  $ git push origin master
  //要推送其他分支，比如dev
  $ git push origin dev
  ```

  但不是所有分支都要往远程仓库推送

  - master是主分支，因此时刻要与远程同步
  - dev是开发分支，团队所有成员都在上面工作，所以也要同步
  - bug分支只用于在本地修复bug，一般不需要推送到远程
  - feature分支是否推送，取决于团队成员是否在上面合作开发

- 抓取分支

  多人协作时，大家都会往master分支上推送自己的修改。

  当其他人第一次从远程库克隆时，默认情况下只能看到本地的master分支。如果要在dev分支上开发，就必须创建远程origin的dev分支到本地

  ```
  $ git checkout -b dev origin/dev
  ```

  现在他就可以在dev上继续修改，然后时不时把dev分支push到远程

- 推送冲突

  当同事push了他的提交，而碰巧你也对同样的文件做了修改，并试图推送

  ```
  $ git add env.txt
  $ git commit -m "add new env"
  $ git push origin dev
  xxxxxxx
  ```

  推送失败，因为你的提交和同事的提交有冲突。解决办法就是，先用git pull把最新提交从origin/dev抓下来，然后在本地合并，解决冲突，再推送：

  ```
  $ git pull
  There is no tracking information for the current branch
  xxxxxxx
  ```

  git pull也失败了，原因是没有指定本地dev分支与远程origin/dev分支的链接。设置dev和origin/dev的链接：

  ```
  //格式是 git branch --set-upstream-to=origin/<remote branch> <local branch>
  $ git branch --set-upstream-to=origin/dev dev
  ```

  再pull：

  ```
  $ git pull
  ```

  这回pull成功，但合并有冲突。解决办法与分支管理中的解决冲突完全一样。解决后，提交，再push：

  ```
  $ git commit -m "fix env conflict"
  xxxxxx
  $ git push origin dev
  xxxxxx
  ```

##### rebase

