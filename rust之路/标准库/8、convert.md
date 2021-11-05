# std::convert

类型间相互转换

本模块中的triat提供了类型转换的方法，各种triat的用途：

- `AsRef`引用到引用的转换（代价小）
- `AsMut`  可变到可变的转换（廉价）
- `From`  值到值的转换(消耗资源)
- `Into`  值到值转换为当前crate外部的类型
- `TryFrom`  `TryInto` 行为类似于`From` `Into` 可以在转换可能失败时使用。

> 此模块中的traits通常用作泛型函数的特征边界，从而支持多个类型的参数。

### 泛型实现

- `AsRef` `AsMut`  如果内部是引用，则自动解引用
- `From<U> for T`  <===> `Into<T> for U`
- `TryFrom<U>for T` <===> `TryInto<T>for <U>`
- From 和Into 是相反的