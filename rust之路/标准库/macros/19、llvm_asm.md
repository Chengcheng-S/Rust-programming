# std::llvm_asm!

```rusr
macro_rules! llvm_asm {
    ("assembly template"
                        : $("output"(operand),)*
                        : $("input"(operand),)*
                        : $("clobbers",)*
                        : $("options",)*) => { ... };
}
```

先行版API

LLVM样式的内联程序集



