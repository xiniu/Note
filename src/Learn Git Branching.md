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
