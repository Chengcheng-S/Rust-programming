# std::error

trait working with Errors

### Traits

Error ：  表示error值得基本期望值的一种trait，即Result<T,E>中的E类型值，

Error 必须通过“Display” 和“Debug” trait进行描述，并可能提供原因链信息。

