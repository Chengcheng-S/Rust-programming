# std::process

处理进程的模块

这个模块主要关注子进程的生成和交互，但是它也提供了`abort`和`exit`来终止当前进程。

### 生成进程

`Command` 结构用于配置和生成进程

```rust
use std::process::Command;
let output=Command::new("echo").arg("hello")
					.output()
					.expect("Failed to execute command");
```

> Command上的几个方法（如spawn或output）可用于生成进程。尤其是，output派生子进程并等待进程终止，而spawn将返回一个子进程，该子进程表示派生的子进程。

### Handle  I/O

子进程的stdout、stdin和stderr可以通过在命令上向相应的方法传递Stdio来配置。一旦生成，就可以从子对象访问它们。

```rust
use std::process::{Command, Stdio};

let echo_child = Command::new("echo")
    .arg("Oh no, a tpyo!")
    .stdout(Stdio::piped())
    .spawn()
    .expect("Failed to start echo process");

let echo_out = echo_child.stdout.expect("Failed to open echo stdout");

let mut sed_child = Command::new("sed")
    .arg("s/tpyo/typo/")
    .stdin(Stdio::from(echo_out))
    .stdout(Stdio::piped())
    .spawn()
    .expect("Failed to start sed process");

let output = sed_child.wait_with_output().expect("Failed to wait on sed");
assert_eq!(b"Oh no, a typo!\n", output.stdout.as_slice());
```

注意ChildStderr和ChildStdout实现Read，ChildStdin实现Write

### Functions

| abort | 以异常的方式终止进程                         |
| ----- | -------------------------------------------- |
| exit  | 使用指定的退出代码终止当前进程               |
| id    | 返回与此进程关联的操作系统分配的进程标识符。 |

### Structs

| Child       | 正在运行或已退出的子进程的表示                               |
| ----------- | ------------------------------------------------------------ |
| ChildStderr | 子进程stderr的句柄                                           |
| ChildStdin  | 子进程的标准输入的句柄                                       |
| ChildStdout | 子进程的标准输出的句柄                                       |
| Command     | 进程生成器，提供了对进程的控制                               |
| ExitStatus  | 描述进程终止后的结果。                                       |
| Output      | 完成的过程的输出                                             |
| Stdio       | 描述当传递给命令的stdin、stdout和stderr方法时，如何处理子进程的标准I/O流。 |
| ExitCode    | 实验： 此类型表示进程在正常终止情况下可以返回其父进程的状态代码。 |

