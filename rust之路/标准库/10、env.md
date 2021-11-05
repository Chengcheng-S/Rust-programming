# std::env

进程环境的检查和操作

此模块包含用于检查各个方面的函数，例如环境变量、进程参数、当前目录和其他各种重要目录。

> 此模块中有多个函数和结构具有以os结尾的对应函数和结构。那些以os结尾的将返回一个OsString，而那些没有的将返回一个字符串。

### Modules

consts 与当前目标关联的常量

### structs

Args        进程参数上的迭代器，为每个参数生成一个字符串值

ArgsOs    进程参数上的迭代器，为每个参数生成一个Os String

JoinPathsError   对路径变量进行操作的错误类型，可能从`env::join_paths`函数返回

SplitPaths      一种迭代器，根据特定于平台的约定将还击你个变量拆分为路径

Vars           此进程的环境变量简介上的迭代器

VarOs        此进程的环境变量简介上的迭代器

### Enums

VarError   与环境变量交互的操作的错误类型，可能从`env::var`函数返回

### Functions

args   返回启动此程序时使用的参数(通常通过命令传递)

args_os   返回启动此程序时使用的参数(通常以命令行传递)

current_dir    以PathBuff形式返回当前工作目录

current_exe   返回当前正在运行的可执行文件的完整的文件路径

home_dir       在已知前提下，返回当前用户主目录的路径(不推荐使用)

join_paths      为PATH环境变量适当的连接一个路径集合

remove_var     从当前正在运行的进程的环境中删除环境变量

set_current_dir    将当前工作目录更改为指定路径

set_var       将环境变量k设置为当前正在与逆行的进程值v

split_paths    根据PATH环境变量的平台约定分析输入

temp_dir        返回临时目录的路径

var                从当前进程获取环境变量键

var_os         从当前进程获取环境变量键，如果未设置该变量，则返回None

vars              为当前进程的所有环境变量返回字符串对(value,key)的迭代器

vars_os         对于当前进程的所有环境变量，返回操作系统字符串对(value,key)对的迭代器
























