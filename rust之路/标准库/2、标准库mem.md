# mem

处理内存的基本函数

此模块包含用于查询类型的大小和对齐方式、初始化和操作内存的函数。

### structs

| Discriminant | 表示枚举的判别式的不透明类型                                |
| ------------ | ----------------------------------------------------------- |
| ManuallyDrop | 禁止编译器自动调用T的析构函数的包装器。这个包装是零成本的。 |

### functions

| align_of                | 返回ABI要求类型的最小对齐                            |
| ----------------------- | ---------------------------------------------------- |
| align_of_val            | 返回val指向的值类型的ABI所需的最小对齐方式。         |
| discriminant            | 返回一个唯一标识符，v中枚举变量的值                  |
| drop                    | 处理(手动销毁)数值                                   |
| forget                  | 获取所有权，并在不运行析构函数的情况下`forget`值     |
| needs_drop              | 若删除的T类型的值很重要，则返回`true`                |
| replace                 | 将src的引用移动到dest中，返回上一个dest的值          |
| size_of                 | 返回类型的大小，单位字节                             |
| size_of_val             | 返回指向值得大小  单位字节                           |
| swap                    | 交换两个可变位置得值，而不取消其中任何一个得初始化。 |
| take                    | 将dest替换为默认值T，返回上一个dest值                |
| transmute(unsafe)       | 将一种类型的值的位重新解释为另一种类型               |
| transmute_copy(unsafe)  | 将src解释为具有类型&U，然后读取src而不移动包含的值。 |
| zeroed(unsafe)          | 返回由全零字节模式表示的T类型的值。                  |
| 以下为先行版 不安全方法 |                                                      |
| align_of_val_raw        | 返回val指向的值类型的ABI所需的最小对齐方式           |
| forget_unsized          | 类似于forget ，同时也接收无大小得值                  |
| size_of_val_raw         | 返回指向值得大小， byte为单位                        |
| variant_count           | 返回枚举类型T中得变量数。                            |
|                         |                                                      |
|                         |                                                      |

### unions

MaybeUninit         构造未初始化的T实例的包装器类型。

