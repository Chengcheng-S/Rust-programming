# std::asm

```
macro_rules! asm{
	("assembly template",
	$(operands,)*
	$(options($(option),*)?
	)=>{...};
}
```

先行版API

内联程序集