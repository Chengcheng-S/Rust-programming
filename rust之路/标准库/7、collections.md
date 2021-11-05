# std::collections

## 集合类型

Rust的集合类型分为：

- Sequences: `Vec` `VecDeque`,`LinkedList`
- Maps:  `HashMap`,`BTreeMap`
- Sets: `HashSet`,`BTreeSet`
- Misc:  `BinaryHeap`

## 使用场景

### Vec

- 收集要在以后处理或发送到其他地方的项，而不关心存储的实际值的任何属性。
- 需要一个按特定顺序排列的元素序列，并且只会附加到（或接近）结尾。
- 存储于堆
- 动态数组
- 堆分配的数组

### VecDeque

- Vec 支持两端输入
- 需要一个队列
- 双端队列

### LinkedList

- 未知大小的Vec VecDeque，不允许摊销
- 高效地分割和追加列表
- 双链表

### HashMap

- 任意值关联键
- 缓存
- map

### BTreeMap

- 按键值排序的map
- 获取一系列的条目
- 找到比某物小或大的最大或最小的key
- 键极值

### map集合变量

- 已存在的key
- 没有与密钥关联的有意义的值。
- 仅需要集合

### BinaryHeap

- 优先队列
- 希望存储一组元素，但只希望在任何给定的时间处理“最大”或“最重要”的元素

## 迭代器

迭代器是Rust标准库中使用的一种强大而健壮的机制。迭代器以通用、安全、高效和方便的方式提供一系列值。迭代器的内容通常是延迟计算的，因此只有实际需要的值才会实际产生，而不需要进行分配来临时存储它们。迭代器主要使用for循环来使用，尽管许多函数在需要值的集合或序列的地方也使用迭代器。

几乎每个集合类型都应该提供的三个主要迭代器是iter、iter_mut和into_iter

- iter提供了一个迭代器，该迭代器以最“自然”的顺序引用集合的所有内容。对于像Vec这样的序列集合，这意味着项目将从0开始按索引的递增顺序生成。对于BTreeMap这样的有序集合，这意味着将按排序顺序生成项。对于HashMap这样的无序集合，将以内部表示最方便的任何顺序生成项。

  ```rust
  let vec=vec![1,2,3,4];
  for x in vec.iter(){
  	println!("{}",x);
  }
  ```

  

- iter_mut以与iter相同的顺序提供可变引用的迭代器。这对于修改集合的所有内容非常有用。

  ```rust
  let mut vec = vec![1, 2, 3, 4];
  for x in vec.iter_mut() {
     *x += 1;
  }
  ```

- into-iter将实际集合按值转换为其内容的迭代器。使用extend with into-iter是将一个集合的内容移动到另一个集合的主要方式。extend会自动调用unuiter，并将任何T:带入迭代器。对迭代器本身调用collect也是将一个集合转换为另一个集合

  ```rust
  let mut vec1 = vec![1, 2, 3, 4];
  let vec2 = vec![10, 20, 30, 40];
  vec1.extend(vec2);
  
  use std::collections::VecDeque;
  
  let vec = vec![1, 2, 3, 4];
  let buf: VecDeque<_> = vec.into_iter().collect();
  ```

- 迭代器还提供一系列适配器方法，用于对序列执行公共线程。适配器中有一些功能性函数，比如map、fold、skip和take。 rev(饭庄任何支持此操作的迭代器)

> 其他一些集合方法也返回迭代器以产生结果序列，但避免分配整个集合来存储结果。这提供了最大的灵活性，因为如果需要，可以调用collect或extend将序列“管道”到任何集合中。否则，序列可以用for循环来循环。迭代器也可以在部分使用后丢弃，从而阻止未使用项的计算。

### entry

entry 根据是否存在键有条件地操纵map的内容，

​	`map.entry(&key)`,map将搜索该键，然后生成Entry enum 一个变体。

> 生成一个`Vacant(entry)`，那么就找不到密钥。在这种情况下，唯一有效的操作是在条目中插入一个值。完成此操作后，将使用空条目并将其转换为对插入的值的可变引用。

> 若释放了一个Occupied(entry)，那么就找到了密钥。在这种情况下，用户有几个选项：他们可以获取、插入或删除被占用条目的值。此外，它们可以将被占用的条目转换为对其值的可变引用，从而为空的insert case提供对称性。







