# std::raw

测试版

包含编译器内置类型布局的结构定义。

可以作为不安全代码中转换的目标，直接操作原始表示。

它们的定义应该始终与`rustc_middle::ty::layout`中定义的ABI相匹配

### structs：

TraitObject       特征对象的表示类似于`&dyn SomeTrait`

