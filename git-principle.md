有这么一个段子...
>老杨一推代码， 所有开发的人便都看着他笑， 有的叫道，“ 老杨， 你又把代码合丢了！” 他不回答， 对产品说，“ 再追加两个需求, 我只做性能优化。” 便排出一台MacBook Pro。 他们又故意的高声嚷道，“ 你推错分支了！” 老杨睁大眼睛说，“ 你怎么这样凭空污人清白……”，“ 什么清白？ 我前天亲眼见你写Bug把流水线搞挂， 又改崩生产代码“。 老杨便涨红了脸， 额上的青筋条条绽出， 争辩道，“ 流水线报错不算错， 合代码挂掉的事能叫Bug吗？” 接连便是难懂的话， 什么“ Merge大法好, GitFlow卍解”， 什么“ 拉代码不需要Rebase” 之类， 引得众人都哄笑起来： 店内外充满了快活的空气。
>
**用git的， 谁没合丢过代码呢？**（ 反正我是合丢过。。。）

好， 痛定思痛， 我决定从原理上理解Git， 所以决定写下这篇文章（ 才不会说是因为buddy叫我准备一场针对Git的session）.
PS： 这篇文章不会是一个体系完善的Git教程， 而是会针对性地从原理上理解一些我们常用的Git命令， 最终目标就是能够在多人合作的场景下保证所有分支的代码安全。

# Git的那些概念
## Git的本质是什么 ?
Git的本质其实是~~复读机~~一个**内容寻址（content-addressable）文件系统**，并在此之上提供了一个版本控制系统的用户界面。Git的核心部分是一个简单的**键值对数据库**。 你可以向该数据库**插入任意类型的值（object）**，它会**返回一个键值（object的引用地址，SHA-1字符串）**，通过该键值可以在任意时刻再次**检索（retrieve）**该内容，因此我们引出了如下的
## 几个重要的概念（类比字典）
- object：字典项
  - blob object：对应的值是特定文件的特定版本的内容。
    ![blob object](http://my.csdn.net/uploads/201206/19/1340112751_1500.jpg)
  - tree object：对应的值的内含信息是一个或多个字典项的数据集，这些被指向项上存放内容可以是`blob object`，也可以是另一个`tree object`。
    ![tree object](http://my.csdn.net/uploads/201206/19/1340112774_4979.jpg)
  - commit object：对应的值的是一个包括一个`tree object`的地址、一个父级`commit object`的地址、一个作者信息以及一个提交message信息）的数据集。
    ![commit object](http://my.csdn.net/uploads/201206/19/1340112824_8482.jpg)
  - tag object：对应的值是一个包括一个地址（99.999%的情形是`commit object`的地址，其他情况不讨论）、一个标签创建者信息、一个日期、一段注释信息的数据集。
    ![tag object](http://my.csdn.net/uploads/201206/19/1340112881_3795.jpg)
所以最终的整体结构应该是这样：
![objects](https://git-scm.com/book/en/v2/images/data-model-3.png)

- ref：指向字典键的引用
  - branch ref：
  - HEAD ref