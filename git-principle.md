有这么一个段子...
>老杨一推代码,  所有的开发同事便都看着他笑,  有的叫道, “ 老杨,  你又把代码合丢了！” 他不回答,  对产品说, “ 追加两个需求, 只做性能优化。” 便排出一台MacBook Pro。 他们又故意的高声嚷道, “ 你推错分支了！” 老杨睁大眼睛说, “ 你怎么这样凭空污人清白……”, “ 什么清白？ 我前天亲眼见你写Bug把流水线搞挂,  又改崩生产代码“。 老杨便涨红了脸,  额上的青筋条条绽出,  争辩道, “ 流水线报错不算错,  合代码挂掉的事能叫Bug吗？” 接连便是难懂的话,  什么“ Merge大法好, GitFlow卍解”,  什么“ 拉代码不需要Rebase” 之类,  引得众人都哄笑起来:  店内外充满了快活的空气。
>
**用 git 的,  谁没合丢过代码呢？**(反正我是合丢过。。。)

好,  痛定思痛,  我决定从原理上理解Git,  所以决定写下这篇文章(才不会说是因为buddy叫我准备一场针对 git 的 session).
PS:  这篇文章不会是一个体系完善的Git教程,  只会针对性地根据某些命令从原理上理解一些我们常用的Git命令, 最终目标就是能够在多人合作的场景下保证所有分支的代码安全。

# 目录
- [目录](#目录)
- [Git的那些概念](#git的那些概念)
  - [Git的本质是什么?](#git的本质是什么)
  - [Git中存储的对象](#git中存储的对象)
    - [1. blob对象](#1-blob对象)
    - [2. tree对象](#2-tree对象)
    - [3. commit对象](#3-commit对象)
    - [**ref(引用)**](#ref引用)
- [Git的那些常用操作](#git的那些常用操作)
  - [file-level](#file-level)
  - [commit/branch-level](#commitbranch-level)
  - [repo-level](#repo-level)
- [Git的那些冲突](#git的那些冲突)
  - [出现冲突的可能情形](#出现冲突的可能情形)
  - [规避冲突的优选操作](#规避冲突的优选操作)
  - [解决冲突的原则](#解决冲突的原则)
  - [解决冲突的优选流程](#解决冲突的优选流程)
  - [其他的奇技淫巧](#其他的奇技淫巧)
    - [别名](#别名)

# Git的那些概念

## Git的本质是什么?

Git的本质其实是~~复读机~~一个**内容寻址（content-addressable)文件系统**<!-- 简单来说就是字典 -->, 它的核心部分是一个简单的**键值对数据库**。我们可以向该数据库**写入值（object)**, 它会**返回一个键(object的引用地址, SHA-1字符串)**, 通过该键可以在任意时刻再次检索值. 我们所操作的版本控制, 其实就是不断地向这个文件系统写入操作日志.

那么这些被写入的操作日志是什么呢?

## Git中存储的对象

###  1. blob对象
对应的值是单个文件内容变动的快照, 这个快照不包括文件名。

![blob](http://my.csdn.net/uploads/201206/19/1340112751_1500.jpg)

###  2. tree对象
以树的形式记录的目录结构和文件的索引, 每个普通结点(有子级的结点)都是一个子级的`tree`对象的包裹体, 每个叶子结点(无子级的结点)都是一个`blob`对象的包裹体, 这些包裹体会附带文件（夹)的名称等元数据。

![tree](http://my.csdn.net/uploads/201206/19/1340112774_4979.jpg)

###  3. commit对象
对应的值包含一个数据集（通常称为`Comments`, 这个数据集包含父级`commit`的地址(SHA1值, 通常也被称作commitid)、作者以及提交message等信息)以及一个当前`commit`对应的变更的`tree`, `tree`的内容为当次提交的变动快照。

![commit](http://my.csdn.net/uploads/201206/19/1340112824_8482.jpg)

总结一下, 三者关系: 

![blob&tree&commit](https://upload-images.jianshu.io/upload_images/3789468-be2115feda33a733.jpg)

所以, 当存在多个提交时,

```bash
#bash
git init
echo "version 1" > test.txt
git add .
git commit -m "frist commit"

echo "version 2" > test.txt
echo "new file" > new.txt
git add .
git commit -m "second commit"

mkdir bak
echo "version 1" > bak/text.txt
git add .
git commit -m "third commit"
```

最终的整体结构应该是这样: 

![objects](https://git-scm.com/book/en/v2/images/data-model-3.png)

<!-- 比较发现, 第1,3次提交的test.txt对应的index(sha1)值一样 -->

### **ref(引用)**

- branch: 指向某一系列提交之首的引用<!-- 分支严格来说只是一个指向commit的引用, 但是git的内建机制在进行相关操作时会针对分支这一类型作特殊的处理 -->
- HEAD: 指向目前工作基点提交的引用<!-- HEAD必须指向分支(symbolic reference)或直接指向提交(detached HEAD), 这里不进行过多发散 -->
- tag: 
  - lightweight tag: 指向任意提交的引用
  - annotated tag: 指向一个标签对象的引用(注:标签对象其实和上面提到的三种一样也是对象, 不过非常罕见, 结构类似commit, 不过内含数据不是指向变动快照的`tree`, 而是指向`commit`)
  <!-- tag对象不做发散 -->
- remote branch: 指向某服务器端某一系列提交之首的引用

# Git的那些常用操作

## file-level

![worktree](https://user-images.githubusercontent.com/11873100/50383511-2be73080-06f0-11e9-8c03-c4017e216875.png)
<!-- 
    1. 不处于git文件系统中的文件处于untracked状态, 已经被提交过但是没有变更过的内容处于unmodified状态, 已经被提交过且更过的内容处于unmodified状态
    2. staged状态的内容会被写入到当次提交的tree中
 -->

  - add @path: 标记文件的stage状态
  - reset @path: 取消文件的stage状态
    - --hard: 取消文件的stage状态并恢复其到unmodified状态
  - checkout @path: 等价于reset @path --hard

## commit/branch-level

**注: 以下的所有commitid都可以换成branch(或者说ref)**<!-- 更能理解其实ref就是对某个commitid的引用 -->

以下一些操作可以到https://learngitbranching.js.org/?NODEMO上进行交互式演示

```bash
git clone # 初始化模拟仓库
```

  - checkout @commitid=null: 转移HEAD到指定提交<!-- commitid为null时等价于 hard reset -->

```bash
git checkout c0 # 将HEAD移动到c0(master保持不变, 此时HEAD为detached HEAD, 直接指向了commit而非某个ref)
```

  - commit: 提交<!-- -m "message"来附加提交信息 -->
    - --amend: 与前一个提交合并提交（改写)

```bash
git commit #生成一个新提交
git commit --amend #改写基点提交, 当前worktree内容融合基点提交的内容重新生成一个提交
```

- branch: 创建一个分支<!-- 实际上是给一个commit打上分支的标记 -->

```bash
git branch branch1 # 基于当前生成一个新分支
git checkout branch1 # 移动HEAD到这个新分支(branch1)
```

- reset @commitid=null: 重置HEAD及其所指向的ref到指定提交<!-- commitid为null时其实操作只是重置了HEAD -->
  - --hard: 抛弃reset过程中的所有文件变更()

```bash
git reset master # 重置HEAD及其所指向的ref到master所在提交
# 重置HEAD及其所指向的ref到"c2'"提交
git reset c2'
```

PS: 关于reset和check的区别
![reset vs checkout](https://user-images.githubusercontent.com/11873100/50234698-3902d780-03f1-11e9-9b6b-28088db3b7a6.png)

  - revert @commitid: 提交一次与某次提交的内容完全相反的提交<!-- 历史依然单向移动 -->

```bash
# 生成一次和c2'相反的提交(抵消/还原c2'提交并向前移动HEAD, 新版本的git还支持批量revert(git revert start..end)
git revert c2'
```

  - cherry-pick @commitid: 将某个commit的变动叠加到HEAD所指向的commit上(会创建一个和之前的commit内容一样的提交, 不过两者具有不同的sha1值)

```bash
git cherry-pick c1 # 将c1提交的变动快照同步到当前HEAD位置, 新版本的git还支持批量cherry-pick(git cherry-pick start..end)
```

  - tag: 标记具有某种特殊意义的提交（里程碑)

```bash
git tag OnMerge # 给当前提交取名"OnMerge", 可以很方便地回滚到特定版本
```

  - merge @commitid: 将某一次提交的内容整合到HEAD(生成一次merge提交叠加在HEAD之上并移动HEAD)

```bash
git checkout master # 模拟其他人在master上进行了两次提交
git commit # 模拟一次提交c3
git commit # 再模拟一次提交c4
git checkout branch1 # 切回HEAD到branch1
git merge c3 # 单独合并c3的变动到当前HEAD
git reset c1'
git merge master # 合并master上所有的变动到当前HEAD
```

  - rebase @commitid: 将某一次提交的内容整合到HEAD(把所有不在commitid对应提交所在提交链上的提交叠加在该提交上)
    - -i 交互式, 可以选择哪些提交要被rebase到指定位置后

```bash
git checkout master # 模拟其他人在master上进行了两次提交
git commit # 模拟一次提交c7
git commit # 再模拟一次提交c8
git checkout branch1 # 切回HEAD到branch1
git rebase c7 # 把当前分支的变更叠加到c7上
git rebase OnMerge # 撤销一下
git rebase master -i # 把当前分支的变更叠加到master上(效果等效于merge, 不过rebase假定自己的编辑都是基于master的)
```

```bash
# 如果经常保持rebase, 那么当branch1需要被merge回master时, 会非常容易
git checkout master
git merge branch1 # 冲突已经在频繁的rebase中被解决的, 这里的merge都会是"快速前进"
# 可以说是多分支开发中的"单分支开发"
```

## repo-level

  - fetch: 同步remote repo的状态<!-- 关于pull, git pull其实是git fetch + get merge <origin名>/<分支名>的组合命令, 在我们trunkflow中, 我们应该使用 --rebase -->
  - remote: 管理remote repo及相关分支, 几乎只需要单次配置, 不做赘述.
  - push: 将本地的commit推送到服务器（默认不会推送tag)<!-- 需要保证此时代码为最新版本 -->
    - push --tag: 连tag一起推送<!-- force 这些操作就不提了, 因为理论来说不太应该使用它们 -->

![image](https://user-images.githubusercontent.com/11873100/50385717-df621c00-0714-11e9-9664-fa4cb17597f9.png)
<!-- 
    1. 本来不存在pull这种操作, 它只是fetch+merge的语法糖
    2. merge和rebase会根据repo的index来更新当前的HEAD
 -->

# Git的那些冲突

## 出现冲突的可能情形

- 他人和自己编辑了对同一文件的编辑内容存在交叉行<!-- 最简单, 借助diff工具可以直接解决 -->
  <!-- 后面两种会比较麻烦, 后面解决冲突部分详细说明 -->
- 他人删除、移动了自己编辑的文件
- 他人和自己新增了同名的文件

## 规避冲突的优选操作

- 代码稳定后尽快推送 <!-- 增加上传频率, 小伙伴们更容易获取到最新的代码 -->
- 勤获取代码（*Build前自动获取最新代码*)<!-- 增加下载频率, 更容易获取到最新的代码 -->
- 结构变更优先提交, 并及时在团队内同步信息 <!-- 避免小伙伴们在已经不存在的文件上编码 -->

<!-- 冲突不可避 -->
## 解决冲突的原则

- 确保现有代码安全
- 叠加新增修改
- 不盲目操作

Then

## 解决冲突的优选流程

1. 使用他人的代码(如果有stash代码, 请务必注意pop出来的代码是自己的代码, 使用gui的时候很容易误解)
2. 叠加自己的改动
3. 遇到一时无法解决的问题, 和原有代码作者pair解决
4. 找不到原有代码作者时可以暂时abort

PS: 针对树冲突（上面可能出现的冲突的后两种)的处理流程（个人总结, 有更好的办法请务必告诉我)
1. abort本次rebase
2. 重命名备份冲突文件并amend commit
3. pull&rebase
4. 借助第三方比较工具, 将自己的改动覆盖在被移动了位置的原文件上
5. 再次amend commit

## 其他的奇技淫巧

### 别名

<!-- 简化一些常用的操作 -->
```bash
git config --global alias.co checkout
```
