# Learn Git Branching

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

实际上iu是取出一些列的提交记录，复制他们，然后再另外一个地方逐个放下去。Rebase的优势就是可以创造更加线性的提交历史，如果只使用Rebasse的化，代码的提交历史将会变得异常清晰。`git rebase master`=把当前的工作复制到master；然后再切换到master分支，执行`git rebase bugFix`,移动master分支的引用
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
git rebase -i
git commit --amend
git rebase -i
git branch -f master HEAD
```





