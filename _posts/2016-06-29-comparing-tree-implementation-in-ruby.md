---
layout: post
title: "比较树的Ruby实现"
tags: []
---

# 1. 什么是树
  树是编程当中常见的数据结构，经常被用来表达具有层级关系的集合。维基对树的定义：

  > A tree is a (possibly non-linear) data structure made up of nodes or vertices and edges without having any cycle. The tree with no nodes is called the null or empty tree. A tree that is not empty consists of a root node and potentially many levels of additional nodes that form a hierarchy.

  树有两类基本操作：

  * 树的更新
  * 树的查询


# 2. 比较树的Ruby实现

## 2.1 Ruby典型的树实现

  目前Ruby语言中有4种典型的树实现GEM包：

  * [Acts As Tree](https://github.com/amerine/acts_as_tree)。
  * [Closure Tree](https://github.com/mceachen/closure_tree)。
  * [Ancestry](https://github.com/stefankroes/ancestry)。
  * [Awesome Nested Set](https://github.com/collectiveidea/awesome_nested_set)

## 2.2 实现思路分析

### Acts As Tree
  ActsAsTree是对ActiveRecord的扩展。它采用“Parent-Child”模式，通过添加parent_id列来维护parent–children关系。

  * 高效的树更新
    由于只通过parent_id来维护层级关系，所有对于树的更新只需要调整parent_id的值，非常方便。
    比如A节点是树的根；B、C节点是A节点的子节点，此时B、C节点的parent_id都是A节点。如果要将C节点移到B节点，作为B节点的子节点，那么只需要修改C节点的parent_id为B节点就完成了更新。

  * 低效的树查询
    由于只通过parent_id来维护层级关系，所以要进行查询操作就变得相对复杂。比如A节点是树的根，如果我们需要查询A节点有哪些后代，那么只能通过递归来获得所有后代，递归的次数取决于树的深度，这种查找效率是非常低下的。

### Ancestry
  Ancestry将ActiveRecord Model通过materialised path pattern组织成树形结构。

  * 相对低效的树更新
    它比“Parent-Child”模式要复杂。当添加节点时，需要获得所有祖先的id，并把层级关系路径化。

  * 相对高效的树查询
    由于把层级关系路径化，通过SQL语言的Like可以快速的获取某个节点的后代。

### Awesome Nested Set
  AwesomeNestedSet是“Nested Set(https://en.wikipedia.org/wiki/Nested_set_model)”模式的实现。

  * 低效的树更新
    由于每个节点需要维护左右域值，所以在更新时会导致大量的重新编号，这将严重影响更新效率。

  * 高效的树查询
    由于维护着左右域值，可以通过SQL语句快速的获得某个节点的后代。

### Closure Tree
  ClosureTree通过parent_id和额外的层级表（以hierarchies为后缀的表）来维护层级关系。

  * 相对低效的树更新
    由于需要额外的维护层级表，所以在频繁更新时会比较慢。

  * 相对高效的树查询
    通过额外的层级表可以快速的获得某个节点的后代。

## 2.3 开发生态比较

  截止2016-6-29日Github上相关数据：

  * ActsAsTree:
    Star: 375; Fork: 136; last commit: 2016-1-25

  * Ancestry:
    Star: 2181; Fork: 303; last commit: 2016-6-13

  * Awesome Nested Set:
    Star: 1799; Fork: 402; last commit: 2016-6-2

  * Closure Tree:
    Star: 854; Fork: 121; last commit: 2016-6-22

# 3. 总结

  不同的实现都有各自擅长的方面，可以根据具体的场景来选择适合的实现。如果树的更新很频繁，查询要求不高，那么可以选择ActsAsTree。如果树的更新频率很低，但查询很频繁，可以选择AwesomeNestedSet。如果两者差不多，那么你可以选择相对均衡的Ancestry。ClosureTree由于需要额外的表来维护层级关系，并且也没有特别明显的优势，所以个人不推荐使用。

# 参考文章
  * [wiki定义](https://en.wikipedia.org/wiki/Tree_(data_structure))
  * [http://railsware.com/blog/2013/07/15/storing-tree-structures-in-the-rdbms/](http://railsware.com/blog/2013/07/15/storing-tree-structures-in-the-rdbms/)