## cargo.toml vs cargo.lock

- Cargo.toml 广义上描述项目的依赖关系
- Cargo.lock 包含依赖项的确切信息，由cargo维护，不应手动进行编辑

更新依赖的库

```
cargo update  // 更新所有的依赖
cargo update -p rand // 更新某个库
```

