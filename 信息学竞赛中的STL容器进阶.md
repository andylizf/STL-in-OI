---
title: 信息学竞赛中的STL容器进阶
categories:
  - STL
tags:
  - null
date: 2020-04-07 18:02:56
---

## 概述

在信息学竞赛中，使用STL是高效率的：一方面，STL的高复用性使编写代码的难度大大降低，加快了开发效率；另一方面，STL的许多算法和容器，在编译器的优化下，特别是在允许使用`O2`编译优化的竞赛场景下，运行效率一般更高。

事实上，在信息学竞赛中，学习STL中的概念对使用STL至关重要，否则各种编译错误将使你焦头烂额。本文面向有C++的指针、引用、函数基础的OI选手，前半部分着重介绍了STL中的许多概念，后半部分着重介绍了各类容器的使用方式。容器不尽相同，其成员函数也又多又杂。介绍这些函数的目的不是使你迅速精通标准库，而是使你明白**容器能做什么**，得以在信息学竞赛中对其合理利用。

## 迭代器

### 迭代器

类似指针的对象。



![range-rbegin-rend.svg](http://upload.cppreference.com/mwiki/images/3/39/range-rbegin-rend.svg)

### 迭代器类型

1. 输入迭代器：

   可**读**迭代器指向的元素值。

   输入迭代器上的算法必须是**单趟**算法：一旦自增输入迭代器，则所有其先前的副本都可能失效。

   典型的例子是[`istream_iterator`](https://en.cppreference.com/w/cpp/iterator/istream_iterator)，就像`cin`读字符流。

2. 输出迭代器：

   可**写**迭代器指向的元素值。

   输出迭代器上的算法必须是**单趟**算法：一旦自增输出迭代器，则所有其先前的副本都可能失效。

   典型的例子是[`ostream_iterator`](https://zh.cppreference.com/w/cpp/iterator/ostream_iterator)，就像`cout`写字符流。

3. 向前迭代器：

   可**读写**迭代器指向的元素值。

   不同于输入迭代器和输出迭代器，向前迭代器可以用于**多趟**算法[^1]。

4. 双向迭代器：

   可**双向移动**，即自增和自减的向前迭代器。

5. 随机访问迭代器

   可在常数时间内指向**任何元素**的双向迭代器。

   许多算法要求容器支持随机访问迭代器，比如`std::sort`和`std::binary_search`。

   数组指针满足随机访问迭代器的所有要求。

6. 连续迭代器

   连续迭代器指向的逻辑相邻元素在内存中物理上相邻。

   数组指针满足随机访问迭代器的所有要求。

   标准库中，仅[`vector::iterator`](https://zh.cppreference.com/w/cpp/container/vector)，对于除`bool`以外的`value_type`，支持连续迭代器<font color="#008000">（C++11 前）</font>：


<style>
    td{
        text-align:center;
    }
</style>
<table>
   <tr>
      <td colspan=5>迭代器类型</td>
      <td>支持的操作</td>
   </tr>
   <tr>
      <td rowspan=6>连续迭代器</td>
      <td rowspan=5>随机访问迭代器</td>
      <td rowspan=4>双向迭代器</td>
      <td rowspan=3>正向迭代器</td>
      <td rowspan=2>输入迭代器</td>
      <td>读入</td>
   </tr>
   <tr>
      <td>自增（非多趟）</td>
   </tr>
   <tr>
      <td></td>
      <td>自增（多趟）</td>
   </tr>
   <tr>
      <td colspan=2></td>
      <td>递减</td>
   </tr>
   <tr>
      <td colspan=3></td>
      <td>随机访问</td>
   </tr>
   <tr>
      <td colspan=4></td>
      <td>连续存储</td>
   </tr>
   <tr>
      <td colspan=6>属于以上类别之一并且还满足输出迭代器要求的迭代器称为可变迭代器。</td>
   </tr>
   <tr>
      <td rowspan=2>输出迭代器</td>
      <td rowspan=2 colspan=4></td>
      <td>写</td>
   </tr>
   <tr>
      <td>自增（非多趟）</td>
   </tr>
</table>

## 容器

**容器**是模板类和算法的泛型集合，使程序员轻松实现常见的数据结构。 容器大体可以分为三类：**序列容器**，**关联容器**和**无序关联容器**，每类容器都旨在支持一组不同的操作。

容器管理了为其元素分配的存储空间，并提供成员函数以访问它们，或直接，或通过迭代器。

大多数容器拥有至少几个相同的成员函数，并且共享功能。 哪种容器最适合特定应用场景，不仅取决于所提供的功能，还取决于其针对不同工作负载的效率。

### 特征与通性

对于元素类型为`T`的容器类型`C`的实例`a, b`：

- `C::iterator`：`C`的迭代器类型，必须是正向迭代器。

- `C::const_iterator`：`C`的`const`迭代器类型，是只读的正向迭代器，不可修改迭代器指向的元素。

- `C::value_type`：为`T`。

- 默认构造`C()`，创建一个空的`C`对象。

- 拷贝构造`C(a)``C b = a`，创建一个`a`的拷贝。

- 赋值`a = b`，销毁`a`并将`b`中的每一个元素复制到`a`。

- `a.begin()``a.cbegin()`：返回容器首的迭代器。在`a`非`const`限定时，前者返回`iterator`类型的迭代器，否则返回`const_iterator`类型的迭代器；后者始终返回`const_iterator`类型的迭代器。

- `a.end()``a.cend()`：返回容器的尾后迭代器。在`a`非`const`限定时，前者返回`iterator`类型的迭代器，否则返回`const_iterator`类型的迭代器；后者始终返回`const_iterator`类型的迭代器。

  ![range-begin-end.svg](http://upload.cppreference.com/mwiki/images/1/1b/range-begin-end.svg)

- `a == b``a != b`：当且仅当`a`和`b`的长度相同，且`a`中每一个元素均与`b`中相应的元素相等时，前者为真。后者等价于`!(a == b)`。

- `a.swap(b)``swap(a, b)`：交换`a`和`b`。

- `a.size()`：返回元素个数，等价于`a.end() - a.begin()`。

- `a.empty()`：当且仅当容器为空时为真，等价于`a.begin() == a.end()`。

<font color="#008000">C++11 前</font>标准库对`const_iterator`的支持不够完善，建议均使用`iterator`。

支持以下操作：

- `erase`：从容器移除指定的元素。

  ```c++
  void erase(iterator pos); // 1
  void erase(iterator first, iterator last); // 2
  size_type erase(const key_type& key); // 3
  ```

  从容器移除指定的元素。

  1. 移除位于 `pos` 的元素。

  2. 移除合法范围 `[first; last)` 中的元素。

  3. 移除关键等于 `key` 的元素（若存在一个）。

  指向被擦除元素的引用和迭代器被非法化。其他引用和迭代器不受影响。

  迭代器 `pos` 必须合法且可解引用。

  形式2.实际上是：

  ```c++
  void erase(iterator first, iterator last)
  {
      iterator it = first;
      while(it != last)
          erase(it++);
  }
  ```

  注意`erase(it++)`，将

- `clear`：从容器擦除所有元素。

- `insert`：插入元素到容器，若容器未含拥有等价关键的元素。

  ```c++
  std::pair<iterator, bool> insert(const value_type& value); // 1
  iterator insert(iterator hint, const value_type& value); // 2
  template<typename InputIt> void insert(InputIt first, InputIt last); // 3
  ```

  1. 插入`value`
  2. 插入 `value`，**建议**到尽可能接近`hint` 的位置。
  3. 插入来自范围 `[first, last)` 的元素。 若范围中的多个元素拥有比较等价的关键，则插入哪个元素是待决的。

### 序列容器

序列容器实现了可顺序访问的数据结构，强调各元素在容器内的顺序。

#### `std::vector`



#### `std::deque`



#### `std::queue`



#### `std::list`



### 关联容器

关联容器实现了可以快速查找（对数复杂度）的排序数据结构。

#### `Compare`具名规则

键的唯一性是通过相等关系确定的，$a$ 与$b$相等，当且仅当$!comp(a, b)$且$!comp(b, a)$，其中$comp$函数为`Compare`所定义的排序规则。

#### 特征与通性

关联容器除满足容器的要求外，还提供了以下成员函数：

- `rbegin``crbegin`：返回反向容器首的迭代器。在`a`非`const`限定时，前者返回`iterator`类型的迭代器，否则返回`const_iterator`类型的迭代器；后者始终返回`const_iterator`类型的迭代器。

- `rend``crend`：返回反向容器的尾后迭代器。在`a`非`const`限定时，前者返回`iterator`类型的迭代器，否则返回`const_iterator`类型的迭代器；后者始终返回`const_iterator`类型的迭代器。

  查找：

- `count`：返回键与指定参数相等的元素数，因为此容器不允许重复故为0或1。

- `find`：

- `lower_bound``upper_bound`：

- `equal_range`：

#### `std::set`

```c++
template <
	typename Key,
	typename Compare = std::less<Key>,
	typename Allocator = std::allocator<Key>
> class set;
```

`std::set`是一个关联容器，其中包含`Key`类型的**唯一对象**的有序集合。 使用`Compare`对键进行排序。搜索、删除和插入操作具有对数复杂度。通常使用红黑树实现。

#### `std::map`

```c++
template <
	typename Key,
	typename T,
	typename Compare = std::less<Key>
	typename Allocator = std::allocator<std::pair<const Key, T> >
> class map;
```

`std::map`是一个有序的关联容器，包含具有唯一键的键值对。使用`Compare`对键进行排序。搜索、删除和插入操作具有对数复杂度。通常使用红黑树实现。

除关联容器的要求外，`std::map`还提供了元素访问的成员函数：

- `operator[]`：返回到映射到等于 `key` 的键的值的引用，若**这种键不存在则进行插入**。

  ```c++
  T& operator[](const Key& key);
  ```

- `at`

  返回到拥有等于 `key` 的关键的元素被映射值的引用。若**无这种元素，则抛出[std::out_of_range](https://zh.cppreference.com/w/cpp/error/out_of_range)异常**。

#### `std::multiset`

与`std::set`相比，`std::multiset`允许存在多个相等的键。自<font color="#008000">C++11</font>起，等价的元素按照插入顺序进行排序。

成员函数基本相同，在此不加赘述。

#### `multimap`

与`std::map`相比，`std::multimap`允许存在多个键值对拥有相等的键。自<font color="#008000">C++11</font>起，等价的元素按照插入顺序进行排序。

除不支持成员访问的`operator []`与`at`函数外，其余成员函数基本相同，在此不加赘述。

### 无序关联容器

无序关联容器实现了可以快速查找（均摊常数，最坏线性）的无序（哈希）数据结构。

#### `unordered_set`



#### `unordered_map`



#### `unordered_multiset`



#### `unordered_multimap`



### 容器适配器

容器适配器为顺序容器提供了不同的接口。

#### `std::stack`



#### `std::queue`



#### `std::priority_queue`



### 迭代器无效

只读方法永远不会使迭代器或引用无效。 修改容器内容的方法可能会使迭代器和/或引用无效，如下表所概述。



## 注释与参考

1. https://en.cppreference.com/w/cpp/container

[^1]:关于多趟保证，请参阅https://zh.cppreference.com/w/cpp/named_req/ForwardIterator#.E5.A4.9A.E8.B6.9F.E4.BF.9D.E8.AF.81