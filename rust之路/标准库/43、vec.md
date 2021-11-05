# std::vec

具有堆分配内容的连续可增长数组类型，`Vec<T>`。

Vectors有O(1)索引， 摊余O(1) 、 push O(1)   pop O(1)

vectors 确保他们分配的字节永远不会超过`isize::MAX`

### Structs

| Drain       | `Vec<T>`的迭代器                              |
| ----------- | --------------------------------------------- |
| Intolter    | 从vec中移除的迭代器                           |
| Splice      | 一种用于矢量的拼接迭代器                      |
| Vec         | 连续的可增长数组类型，Vec<T>                  |
| DrainFilter | 实验：  在Vec上调用drain_filter生成的迭代器。 |

