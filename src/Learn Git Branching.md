# Learn Git Branching

https://learngitbranching.js.org/?locale=zh_CN

## Git Commit

Git仓库中提交的记录保存的是所有文件的快照；但是实际内部实现时并不会盲目的复制整个目录，他保存的是差异。Git会保存提交的历史记录。
```
git commit -m someMsg
```

## Git Branch

Git的分支非常轻量，他们只是简单的指向某条记录。**早建分支多用分支，分支就算创建再多也不会造成储存或内存上的开销**。使用分支其实就相当于在说“我想基于这个提交以及它所有的父提交进行新的工作”

```
git branch bugFix
git checkout bugFix
```
有一种更简洁的方式，创建一个新的分支同时切换到新创建的分支的化
```
git checkout -b bugFix
```

## 分支与合并

如何将两个分支合并到一起。第一种方法就是 git merge。在git中合并两个分支时会产生一个特殊的提交记录，他有两个父节点，翻译成自然语言相当于：将两个父节点本身以及他们所有的祖先都包含进来

```
git checkout -b bugFix
git commit -m someMsg
git checkout master
git commit -m someMsg1
git merge bugFix
```

## Git Rebase

实际上iu是取出一些列的提交记录，复制他们，然后再另外一个地方逐个放下去。Rebase的优势就是可以创造更加线性的提交历史，如果只使用Rebasse的化，代码的提交历史将会变得异常清晰。`git rebase master `=把当前的工作复制到master；然后再切换到master分支，执行`git rebase bugFix`,移动master分支的引用,如果跟两个参数,`git rebase A B`就是将B复制到A,你可以理解为B是可选参数
```
git checkout -b bugFix
git commit -m someMsg
git checkout master
git commit -m someMsg2
git checkout bugFix
git rebase master
```

## 在提交树上移动

**HEAD**是一个对当前检出记录的符号引用，总是指向当前分支上最近一次的提交记录。
分离HEAD就是让其指向具体的提交记录而不是分支`git checkout C1`后HEAD指向C1
```
git checkout C4
```

## 相对引用

通过指定的提交记录的哈希值在git中移动不太方便（哈希值很长，需要git log查看记录的哈希值，一般而言你可以只输入哈希值的前几个字符，但还是很麻烦）
- 使用^向上移动1个提交记录，把这个加在引用名称后面标识相当于让git寻找指定提交记录的父提交`git checkout master^`
- 使用~num向上移动多个提交记录,如果不跟数字的话效果跟^类似`git checkout HEAD~4`后退4步
```
git checkout bugFix^
```
- 强制修改分支的位置:将master分支指向HEAD的第3级父提交
```
git branch -f master HEAD~3
```

```
git branch -f master C6
git branch -f bugFix HEAD~2
git checkout HEAD^
```

## 撤销变更

主要有两种方法来撤销变更，一种是git reset，还有就是git revert
- git reset通过把分支记录回退几个提交提交记录来实现撤销改动---“改动历史”；但是这种方式对使用远程分支是无效的
- git revert，撤销修改并可以分享给他们，实际会生成提交记录
```
git reset HEAD^
git checkout pushed
git revet C2
```

## 整理提交记录

### Git cherry-pick

把一些提交复制到当前所在分支。`git cherry-pick C2 C4`将C2 C4提交抓过来放在当前分支下
```
git cherry-pick C3 C4 C7
```
### 交互式的rebase

当前知道提交值的哈希值时git cherry-pick非常好用，但是如果不知道呢？ 可以利用交互式的rebase---`git rebase -i ovherHeae`

### 本地栈式提交

如果我们只想提交解决问题的哪一个提交记录，我们可以使用`git rebase -i`或者`git cherry-pick`
```
git checkout master
git cherry-pick C4
```

### 提交的技巧

```
git rebase -i
git commit --amend
git rebase -i
git branch -f master HEAD
```


### 提交的技巧2

```
git checkout master
git cherry-pick C2
git commit --amend
git cherry-pick C3
```

### git tags

分支容易被人为移动且有新的提交也会移动，很容易改变。有没有什么可以永远指向某个提交的标识呢？比如发布的大版本。git tag就是干这恶鬼的，他们可以创建一个永久的里程碑，不会被改变或者移动.检出到v1上面会进入到分离HEAD的状态
```
git tag v1 C1 # 创建一个V1 tag，指向记录C1
git checkout v1
```

### git describe

git专门设计的命令来描述理你最近的标签，输出的tag_num_g(哈希值或者标签)

### 多分支rebase

`git rebase master `=把当前的工作复制到master；然后再切换到master分支，执行`git rebase bugFix`,移动master分支的引用,如果跟两个参数,`git rebase A B`就是将B复制到A,你可以理解为B是可选参数

```
git rebase master bugFix
git rebase bugfix side
git rebase  side another
git rebase another master
```

### 选择父提交记录

因为合并分支式一个节点可以有多个父提交，^num可以指定式第几个父提交，默认式第一个。~^支持链式操作
```
git checkout HEAD~^2~2
git branch bugWork HEAD^^2^ # 创建分支时可以指定一个位置
```

### 纠缠不清的分支

```
git branch -f three C2
git checkout one
git cherry-pick C4 C3 C2
git checkout two
git cherry-pick C5 C4 C3 C2
```

## git clone

远程仓库只是你的仓库在另一台计算机上的拷贝，你可以通过网络和他通信，他有一些列的强大特性
- 是一个强大的备份
- 让代码社交化，例如github
```
git clone
```

## 远程分支

远程分支反映了远程仓库的状态，远程分支有一个特别的属性是检出时自动进入了分离HEAD状态，让你无法直接在这些分支上操作，即他的更新不会依赖你本地的提交
一般开发人员会将主要的远程操作命名为origin
```
git commit -m someMsg
git checkout o/master
git commit -m someMsg # 这个操作不会更新o/master
```

## git fetch

如何从远程仓获取数据--git fetch。当我们从远程分支获取数据时，远程仓库也会更新
执行git fetch后，将本地没有的提交下载至本地，同时远程分支也会更新。但是本地的master并没有，即他不会改你本地的master分支
```
git fetch url
```

## git pull

但是如何将这些变化更新到我们工作当中呢？ 可以使用如下命令,先抓取再合并：
- git cherry-pick o/master
- git rebase o/master
- git merge

实际上先抓取再合并的流程很常用，因为专门提供了一个命令来完成这两个操作，就是git pull（git fetch和git merge）
```
git clone
git fakeTeamwork 2
git commit -m someMsg
git pull
git pull
```

## git push

发布你的成果。远程仓库接受新的本地提交，远程仓库中master分支也被更新，同时本地的远程分支也会被更新，所有的分支都会被更新
```
git commit -m someMsg
git commit -m someMsg
git push
```

## 偏离的工作

因为某些情况下（历史偏离）有许多的不确定性，git时不允许你push变更的，他会强制你先合并最新的代码才能分享你的工作。
可以在push之前做rebase
```
git fetch
git rebase o/master # 也可以使用 git merge o/master
git push
```

有没有更显的方法？ 有！ git pull --rebase
```
git pull --rebase; git push
```
下面效果是一样的,但是会产生合并节点？
```
git pull; git push
```

## 远程服务拒绝 remote rejected

在一个大的合作团队中，master可能被锁定了，需要 pull request的流程来合并修改。应该按照流程新建一个分支并推送这个分支并申请pull request。但是你直接提交给了master，导致你现在被卡住且无法推送。
你需要：新建分支并推送服务器，然后reset master和远程服务器移植，否则下次pull时如果你和别人提交冲突时会有问题
```
git reset --hard o/master
git checkout -b feature C2 # 可以跟第二个参数
git push origin feature
```

## 合并特性分支

将特性分支集成到master并推送到远程分支:`git rebase A B`就是将B复制到A,你可以理解为B是可选参数(只是接，但不会改变A所指位置，但是会改变B)

```
git fetch
git rebase o/master side1
git rebase side1 side2
git rebase side2 side3
git rebase side3 master
```

## 为什么不用merge呢

rebase的优点：使你的提交树变得干净
rebase的缺点：修改了提交树的历史，例如C1可以被rebase到C之后，看起来时C1在C3之后但实际在C3之前
```
git checkout matser
git pull --rebase
git merge side1
git merge side2
git merge side3
git push
```

## 远程跟踪分支

似乎是：master和o/master是关联的，pull时提交记录会先下载到o/master然后再合并到本地master；puhs时将master推送到远程的master分支同时会更显远程分支o/master

这种设定就是分支的 remote tracking属性决定；克隆仓库时git自动帮你设定了该属性。但是我们也可以自己指定这种跟踪关系，可以让任意分支跟踪o/master，最终该分支会想master分支一样得到隐含的push目的以及merge目的，这意味这你可以再分支totallyNotMaster上执行push并将工作推送到远程master分支，但这种情况master不会被更新了

两种方法：
- git checkout -b totallyNotMaster o/master
- git branch -u o/master foo（如果当前就再foo上可以省略foo）
```
git checkout -b side o/master
git commit
git pull --rebase
git push
```

## git push参数
 git push <remote> <place> 切换到本地的place分支获取所有提交并到远程库remote中master分支，如果不带参数，push实际时将HEAD跟踪的分支提交
  
 ```
 git push origin master
 git push origin side
 
 ```
 
 ## <place>参数详解
  
  在上面的来自中，当place参数指定为master时，他同时指定了提交记录的来源和去向；但是当来源和去向不同呢，例如将本地的foo分支推送到远程的bar分支
`git push prigin <source>:<destination>  `。例如：
```
git push origin foo^:master # 记得push会更显本地的o/master
```
当要推送的目的分支不逊在时，git会在远程仓库根据你提供的名称帮你创建这个分支
```
git push origin master:newBranch # 执行完后会更新o/newBranch
```

## git fetch参数

git fetch的参数和git push完全相似，只是方向相反

- git到远程仓库foo分支获取本地不存在的提交放到本地o/foo上。会自己更新o/foo;但是没有更新本地的foo：
git fetch origin foo
- 使用source:destination，可以直接将代码更新到本地分支，但是这个分支不能时当前检出的;如果当前分支不存在，会创建出分支来
git fetch origin foo~1:bar
- 如果git fetch没有参数，他会下载所有提交记录到各个远程分
git fetch

```
git fecth origin foo:master
git fetch origin master^:foo
git checkout foo
git merge master
```

## 古怪的sorce

有两种关于sorce的用法比较诡异，你可以在使用git push或者给i他fetch时不指定任何source，只保留毛猴和destina

- 如果push 空到远程仓库会删除远程仓库的分支
`git push origin :foo # 会删除远程仓库的foo分支`
- 如果fetch 空到本地，会在本地创建新的分支
`git fetch origin :bar # 本地创建bar分支`




