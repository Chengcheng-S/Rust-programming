# rand

```rust
cargo.toml
.....
rand="0.7.3"
```

随机数生成工具

Rand 提供了生成随机数，将其转换为有用类型和分布的实用程序，以及一些与随机性相关的算法。

### Modules

| distributions | 由概率分布生成随机样本 |
| ------------- | ---------------------- |
| prelude       | 入口                   |
| rngs          | 随机数生成器和适配器   |
| seq           | 序列相关功能           |

### Structs

Error		随机数生成器的错误类型

### Functions

| random     | 使用线程本地随机数生成器生成随机值                           |
| ---------- | ------------------------------------------------------------ |
| thread_rng | 检索由系统设定的延迟初始化的线程本地随机数生成器，用于方法连接样式，(thread_rng().gen::<i32>()) 或本地缓存，(let mut rng=thread_rng();)  由Default trait调用，使ThreadRng::Default 等效 |

### Traits

| AsByteSliceMut | 将类型强制转换为字节片的 trait                               |
| -------------- | ------------------------------------------------------------ |
| CryptoRng      | 标记trait  用于指示RngCore或BlockRngCore实现应该是加密安全的。 |
| Rng            | RngCore上自动实现的扩展特性，为采样值和其他方便的方法提供高级通用方法 |
| RngCore        | 随机数的核心                                                 |
| SeedableRng    | 一个随机数生成器，可以显示的设置种子                         |

