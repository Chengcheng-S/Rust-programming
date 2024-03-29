# std::slice

将数据动态调整为连续序列。

切片是内存块的视图，用指针和长度表示。

```rust
let vec=vec![1,2,3,4];
let int_a=&vec[..];
```

共享切片或可变切片。共享切片类型是&[T]，而可变切片类型是&mut[T]，其中T表示元素类型。 

可以改变可变切片指向的内存块：

```rust
let x = &mut [1, 2, 3];
x[1] = 7;
assert_eq!(x, &[1, 7, 3]);
```

### Trait implementations

切片的公共特征有几种实现:

- `Clone`
- `Eq`   `Ord`    对于元素类型为`Eq` `Ord`的切片。
- `Hash`  对于元素类型为`hash` 的切片。

### Iteration

切片实现到迭代器中。迭代器生成对切片元素的引用

```rust
let numbers = &[0, 1, 2];
for n in numbers {
    println!("{} is a number!", n);
}
```

可变元素即可变引用

```rust
let mut scores = [7, 8, 9];
for score in &mut scores[..] {
    *score += 1;
}
```

- `iter`   `iter_mut`  返回默认迭代器的显式方法。
- 返回迭代器的其他方法 `split`   `splitn`   `chunks` `windows` 

### Traits

| SliceIndex | 用于索引操作的助手trait |
| ---------- | ----------------------- |
| Concat     | [T]::concat 的帮助trait |
| join       | [T]::join 的帮助trait   |

### Functions

| from_mut           | 对T的引用转换为长度为1的切片(不可复制)      |
| ------------------ | ------------------------------------------- |
| from_raw_parts     | 从指针和长度形成一个切片                    |
| from_raw_parts_mut | 返回一个可变切片，执行与raw_parts相同的功能 |
| from_ref           | 将对T的引用转换为长度为1的切片(不可复制)    |

### Structs

| Chunks          | 在（非重叠）块（一次块大小元素）中（非重叠）片上的迭代器，从片的开头开始。 |
| --------------- | ------------------------------------------------------------ |
| ChunksExact     | 在（非重叠）块（一次块大小元素）中（非重叠）片上的迭代器，从片的开头开始 |
| ChunksExactMut  | 在（不重叠）可变块（一次块大小元素）中的片上的迭代器，从片的开头开始。 |
| ChunksMut       | 在（不重叠）可变块（一次块大小元素）中的片上的迭代器，从片的开头开始 |
| Iter            | 不可变的切片迭代器                                           |
| IterMut         | 可变的切片迭代器                                             |
| RChunks         | 一个迭代器，位于（非重叠）块（一次块大小元素）中的一个片上，从片的末尾开始。 |
| RChunksExact    | 一个迭代器，位于（非重叠）块（一次块大小元素）中的一个片上，从片的末尾开始。 |
| RChunksExactMut | 一个迭代器，位于（非重叠）块（一次块大小元素）中的一个片上，从片的末尾开始。 |
| RChunksMut      | 一个迭代器，位于（不重叠）可变块（一次块大小元素）中的一个切片上，从切片的末尾开始。 |
| RSplit          | 一种迭代器，位于由与谓词函数匹配的元素分隔的子片上，从片的末尾开始。 |
| RSplitMut       | 向量子片上的迭代器，这些子片由与pred匹配的元素分隔，从片的末尾开始。 |
| RSplitN         | 子片上的迭代器，由与谓词函数匹配的元素分隔开，从片的末尾开始，限制为给定的拆分数。 |
| RSplitNMut      | 子片上的迭代器，由与谓词函数匹配的元素分隔开，从片的末尾开始，限制为给定的拆分数。 |
| Split           | 由与谓词函数匹配的元素分隔的子片上的迭代器                   |
| SplitMut        | 可替换向量的子元素被匹配的元素分开。                         |
| SplitN          | 由与谓词函数相匹配的元素分隔的子片上的迭代器，限制为给定的拆分数 |
| SplitNMut       | 由与谓词函数相匹配的元素分隔的子片上的迭代器，限于给定的拆分数。 |
| Windows         | 长度大小重叠子空间上的迭代器。                               |
| ArrayChunks     | 实验：在（非重叠）块（一次N个元素）中的一个切片上的迭代器，从切片的开始处开始。 |













