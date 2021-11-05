# std::path

跨平台路径操纵

这个模块提供了两种类型，PathBuf和Path（类似于String和str），用于抽象地处理路径。这些类型是分别围绕OsString和OsStr的**瘦包装器**，这意味着它们根据本地平台的路径语法直接在字符串上工作。

### Constatns

MAIN_SEPARATOR     当前平台的路径组件的主分隔符

### Functions

is_separator        确定字符是否是当前平台允许的路径分隔符之一。

### Enums 

Component          路径的单一组成部分。

Prefix               Windows路径前缀



   