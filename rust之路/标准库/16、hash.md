# std::hash

通用hash支持

此模块提供一种计算值哈希的通用方法。使类型hash的最简单方法是使用`#[derive(Hash)]`：

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};

#[derive(Hash)]
struct Person {
    id: u32,
    name: String,
    phone: u64,
}

let person1 = Person {
    id: 5,
    name: "Janet".to_string(),
    phone: 555_666_7777,
};
let person2 = Person {
    id: 5,
    name: "Bob".to_string(),
    phone: 555_666_7777,
};

assert!(calculate_hash(&person1) != calculate_hash(&person2));

fn calculate_hash<T: Hash>(t: &T) -> u64 {
    let mut s = DefaultHasher::new();
    t.hash(&mut s);
    s.finish()
}
```

### Macros

Hash   派生生成特征哈希的impl的宏。

### structs

| BuildHasherDeafault | 用于为实现Hasher和default的类型创建默认的BuildHasher实例。 |
| ------------------- | ---------------------------------------------------------- |
| SipHasher           | SipHash 2-4的实现                                          |

### Traits

| BuildHash | 用于创建哈希器实例的特性。 |
| --------- | -------------------------- |
| Hash      | Hashable  type             |
| Hasher    | 用于散列任意字节流的特征。 |



