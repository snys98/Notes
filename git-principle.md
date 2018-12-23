有这么一个段子...
>老杨一推代码,  所有的开发同事便都看着他笑,  有的叫道, “ 老杨,  你又把代码合丢了！” 他不回答,  对产品说, “ 追加两个需求, 只做性能优化。” 便排出一台MacBook Pro。 他们又故意的高声嚷道, “ 你推错分支了！” 老杨睁大眼睛说, “ 你怎么这样凭空污人清白……”, “ 什么清白？ 我前天亲眼见你写Bug把流水线搞挂,  又改崩生产代码“。 老杨便涨红了脸,  额上的青筋条条绽出,  争辩道, “ 流水线报错不算错,  合代码挂掉的事能叫Bug吗？” 接连便是难懂的话,  什么“ Merge大法好, GitFlow卍解”,  什么“ 拉代码不需要Rebase” 之类,  引得众人都哄笑起来:  店内外充满了快活的空气。
>
**用git的,  谁没合丢过代码呢？**（ 反正我是合丢过。。。）

好,  痛定思痛,  我决定从原理上理解Git,  所以决定写下这篇文章（ 才不会说是因为buddy叫我准备一场针对Git的session）.
PS:  这篇文章不会是一个体系完善的Git教程,  而是会针对性地从原理上理解一些我们常用的Git命令,  最终目标就是能够在多人合作的场景下保证所有分支的代码安全。

# 目录
- [目录](#目录)
- [Git的那些概念](#git的那些概念)
  - [Git的本质是什么 ?](#git的本质是什么-)
  - [几个重要的对象](#几个重要的对象)
    - [**ref: 引用（没有实际内容, 只是SHA-1的别名）**](#ref-引用没有实际内容-只是sha-1的别名)
- [Git的那些常用操作](#git的那些常用操作)
  - [file-level](#file-level)
  - [commit-level](#commit-level)
  - [repo-level](#repo-level)
- [Git的那些冲突（针对TrunkFlow）](#git的那些冲突针对trunkflow)
  - [出现冲突的可能情形](#出现冲突的可能情形)
  - [规避冲突的优选操作](#规避冲突的优选操作)
  - [解决冲突的原则](#解决冲突的原则)
  - [解决冲突的优选流程](#解决冲突的优选流程)
  - [其他的奇技淫巧](#其他的奇技淫巧)
    - [别名](#别名)
    - [之前了解到的一些关于gitflow的误解](#之前了解到的一些关于gitflow的误解)

# Git的那些概念

## Git的本质是什么 ?

Git的本质其实是~~复读机~~一个**内容寻址（content-addressable）文件系统**, 并在此之上提供了一个版本控制系统的用户界面。Git的核心部分是一个简单的**键值对数据库**。 你可以向该数据库**插入任意类型的值（object）**, 它会**返回一个键值（object的引用地址, SHA-1字符串）**, 通过该键值可以在任意时刻再次检索该内容, 因此我们引出了如下的

## 几个重要的对象

- blob: 对应的值是特定文件的特定版本的内容（单个文件内容快照, 不包括文件名）。

    ![blob](http://my.csdn.net/uploads/201206/19/1340112751_1500.jpg)

- tree: 以树的形式记录仓库的目录结构和文件的索引, 每个普通结点都是一个子级的`tree`对象的包裹体, 每个叶子结点都是一个`blob`对象的包裹体, 这些包裹体会附带文件（夹）的名称等元数据。

    ![tree](http://my.csdn.net/uploads/201206/19/1340112774_4979.jpg)

- commit: 对应的值包含一个数据集（通常称为`Comments`, 这个数据集包含父级`commit`的地址、作者以及提交message等信息）以及一个当前`commit`对应的`tree`(仓库根目录)。

    ![commit](http://my.csdn.net/uploads/201206/19/1340112824_8482.jpg)

以Git GUI里面的一个提交为例, 可以看得更加清楚
    ![image](https://user-images.githubusercontent.com/11873100/50170232-5de04780-032a-11e9-9100-3ce2f3ceba7b.png)
    （红色框部分为`Comments`数据集, 蓝色框部分为指向一个文件的`tree`）

总结一下, 三者关系: 

![blob&tree&commit](https://upload-images.jianshu.io/upload_images/3789468-be2115feda33a733.jpg)

所以最终的整体结构应该是这样: 

![objects](https://git-scm.com/book/en/v2/images/data-model-3.png)

PS: 我们所使用的日常git命令基本都是在操作commit

### **ref: 引用（没有实际内容, 只是SHA-1的别名）**

- branch: 指向某一系列提交之首的引用(和普通使用理解意义上的分支有些区别)
- HEAD: 指向目前工作基点的引用<!-- 通常是分支, 指向某个commit时称为"detached HEAD", 这里不进行发散 -->
- tag: 
  - lightweight tag: 指向任意提交的引用
  - annotated tag: 指向一个标签对象的引用(注:标签对象其实和上面提到的三种一样也是对象, 不过非常罕见, 结构类似commit, 不过内含数据不是指向仓库快照的`tree`, 而是指向`commit`)
  <!-- tag对象不做发散 -->
- remote branch: 指向某服务器端某一系列提交之首的引用

# Git的那些常用操作

![image](https://user-images.githubusercontent.com/11873100/50385111-344c6500-070a-11e9-8ec2-6ba5e60e262c.png)

## file-level

  - add @path: 标记文件需要被提交
  - reset @path: 取消文件的stage状态
    - --hard: 取消文件的stage状态并恢复其到unmodified状态
  - checkout @path: 等价于reset @path --hard

![worktree](https://user-images.githubusercontent.com/11873100/50383511-2be73080-06f0-11e9-8c03-c4017e216875.png)

## commit-level

  - commit: 提交<!-- -m "message"来附加提交信息 -->
    - --amend: 与前一个提交合并提交（改写）
  - reset @commitid=null: 重置HEAD到指定提交<!-- commitid为null时其实操作只是重置了HEAD -->
    - --hard: 抛弃reset过程中的所有文件变更
  - checkout @commitid=null: 转移HEAD到指定提交<!-- commitid为null时等价于reset --hard -->
  - revert @commitid: 提交一次与某次提交的内容完全相反的提交<!-- 历史依然单向移动 -->
  ![reset vs checkout](https://user-images.githubusercontent.com/11873100/50234698-3902d780-03f1-11e9-9b6b-28088db3b7a6.png)
  - tag: 标记具有某种特殊意义的提交（里程碑）
  - branch: 创建一个分支<!-- 实际上是给一个commit打上分支的标记 -->
  - push: 将本地的commit推送到服务器（默认不会推送tag）<!-- 需要保证此时代码为最新版本 -->
    - push --tag: 连tag一起推送<!-- force 这些操作就不提了, 因为理论来说不应该使用它们 -->
  - cherry-pick @commitid: 将某个commit的变动叠加到HEAD所指向的commit上(会创建一个和之前的commit内容一样的提交, 不过两者具有不同的sha1值)

## repo-level
  - fetch: 同步remote repo的状态<!-- 关于pull, git pull其实是git fetch + get merge <origin名>/<分支名>的组合命令, 在我们trunkflow中, 我们应该使用 --rebase -->

# Git的那些冲突（针对TrunkFlow）

## 出现冲突的可能情形

- 他人和自己编辑了对同一文件的编辑内容存在交叉行<!-- 最简单, 借助diff工具可以直接解决 -->
  <!-- 后面两种会比较麻烦, 后面解决冲突部分详细说明 -->
- 他人删除、移动了自己编辑的文件
- 他人和自己新增了同名的文件

## 规避冲突的优选操作

- 小步提交 <!-- 增加上传频率, 小伙伴们更容易获取到最新的代码 -->
- 勤获取代码（*Build前自动获取最新代码*）<!-- 增加下载频率, 更容易获取到最新的代码 -->
- 结构变更优先提交, 并及时在团队内同步信息 <!-- 避免小伙伴们在已经不存在的文件上编码 -->

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

PS: 针对树冲突（上面可能出现的冲突的后两种）的处理流程（个人总结, 有更好的办法请务必告诉我）
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

### 之前了解到的一些关于gitflow的误解

1. gitflow的feature分支允许force push
3. trunkflow, 其实是把feature分支转换成为了"本地分支", 优点是可以在本地安全便捷地使用rebase, 缺点是不便于进行"多任务"
<!-- 故事卡进行中, 需要临时处理优先级更高的卡, gitflow可以在feature分支上提交并push并妥善保存, trunkflow似乎就只有修复代码至不影响其他功能或把本机commit作为patch脱离git保存(或者reset&stash) -->
4. gitflow, feature分支在合并在dev之前也可以(在比较常见的情况下,也应该)rebase到dev的最新提交上, 不一定要进行大量的merge