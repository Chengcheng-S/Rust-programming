# std::time



时间量化

eg：

```rust
use std::time::Duration;
let a=Duration::new(5,0);
assert_eq!(Duration::new(5, 0), Duration::from_secs(5));
```

### Constants

UNIX_EPOCH       时间锚，用于创建新的System Time实例或了解System Time所在的时间位置

### Structs

| Duration        | 时间跨度的持续时间类型，常用于系统超时                       |
| --------------- | ------------------------------------------------------------ |
| Instant         | 对单调不变时钟的测量。不透明，仅对持续时间有用。             |
| System Time     | 系统时钟的一种度量，用于与外部实体（如文件系统或其他进程）对话 |
| SystemTimeError | 从系统时间上的duration_since和elapsed方法返回的一种错误，用于了解系统时间的反向距离。 |

