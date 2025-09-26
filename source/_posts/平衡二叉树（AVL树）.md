---
title: 平衡二叉树（AVL树）
categories:
  - 数据结构
description: 平衡二叉树（AVL树）
date: 2025-08-24 14:50:15
tags:
---

## 平衡二叉树的基础概念

平衡二叉树 全称叫做 平衡二叉搜索（排序）树，简称 AVL树。英文：Balanced Binary Tree （BBT），注：二叉查找树(BST)

**AVL什么意思**  
AVL 是大学教授 G.M. Adelson-Velsky 和 E.M. Landis 名称的缩写，他们提出的平衡二叉树的概念，为了纪念他们，将 平衡二叉树 称为 AVL树。

AVL树本质上是一颗二叉查找树，但是它又具有以下特点：

- 它是一棵空树或它的左右两个子树的高度差的绝对值不超过1，
- 左右两个子树 也都是一棵平衡二叉树。

在AVL树中，任何节点的两个子树的高度最大差别为 1 ，所以它也被称为平衡二叉树 。

### 平衡因子（Balance Factor，简写为bf）

**概念**  
结点的左子树的深度减去右子树的深度。  
即： 结点的平衡因子 = 左子树的高度 - 右子树的高度 。

在 AVL树中，所有节点的平衡因子都必须满足： -1<=bf<=1;

## 参考

[CSDN_数据结构 —— 图解AVL树(平衡二叉树)](https://blog.csdn.net/xiaojin21cen/article/details/97602146)