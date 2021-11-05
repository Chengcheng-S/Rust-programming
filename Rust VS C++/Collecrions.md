# Collections

集合是以某种方式容纳零个或多个元素的合集，允许枚举这些元素、添加、删除、查找等操作。

- Vector ：动态数组，从末端符间或删除元素的成本很低，插入向或从数组的任何其他部分移除项的成本更高，并且设计内存移动。重新分配内存可能代价更高，甚至导致碎片化。
- Veddeque：环形缓冲数组。 可以以低成本的方式从两端添加或者删除项目。数组中的元素不是按照顺序排列的，因此管理和移除数据比较复杂一些。
- LinkList：链表 为每个元素单独分配内存，因此从列表中的任何位置添加或删除元素都很便捷成本低，到那时，按索引迭代链表的开销会很大，导致堆分配的内存很大。
- Set ：包含唯一元素的集合，插入重复的项不会成功。有些集合保持插入顺序，若要去重，这十分有效。
- Map：k-v存储的集合，键的唯一性。

| C    | C++         | Rust                                                   |
| ---- | ----------- | ------------------------------------------------------ |
|      | std::vector | std::vec::Vec   std::collcetions::VecDeque             |
|      | std::list   | std::collections::LinkedLIst                           |
|      | std::set    | std::collections::HashSet   std::collections::BTreeSet |
|      | sdt::map    | std::collections::HashMap  std::collections::BTreeMap  |

C没有标准的集合类型，有些库提供glib或cii等api

## Iterators

迭代器是对集合中某个位置的引用，用于一次遍历一个元素。

### C++

C++提供了一个迭代集合的快捷方式

```c++
std::vector<char>values;

for(const char &c :values){
	//do somethings
}
```



C++11

```c++
std::vector<char> values;
for (std::vector<char>::const_iterator i = values.begin(); i != values.end(); ++i) {
    const char &c = *i;
    // do something to process the value in c
}
```

Rust也有迭代器，它们以类似的方式工作，通过C++来增加它们通过集合的方式。