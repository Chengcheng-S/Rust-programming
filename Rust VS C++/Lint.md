## Lint

C/C++编译器可以发出许多警告，但是静态分析的数量通常是有限的。

Rust编译器对数据执行更严格的生命周期的检查，然后执行lint检查，检查代码是否存在潜在的错误。

- Dead / unused code
- Unreachable code
- Deprecated methods
- Deprecated methods
- Camel case / snake case violations
- Unbounded recursion code (i.e. no conditionals to stop recursion)
- Use of heap memory when stack could be used
- Unused extern crates, imports, variables, attributes, mut, parentheses
- Using “while true {}” instead of “loop {}”

使用以下的属性，可以更严格地实施或忽略Lint规则：

```rust
#[allow(rule)]
#[warn(rule)]
#[deny(rule)]
#[forbid(rule)]
```

更对规则可以使用`rustc -W help` 查看

```
Lint groups provided by rustc:

                       name  sub-lints
                       ----  ---------
                   warnings  all lints that are set to issue warnings
        future-incompatible  keyword-idents, anonymous-parameters, illegal-floating-point-literal-pattern, private-in-public, pub-use-of-private-extern-crate, invalid-type-param-default, safe-packed-borrows, patterns-in-fns-without-body, missing-fragment-specifier, late-bound-lifetime-arguments, order-dependent-trait-objects, coherence-leak-check, tyvar-behind-raw-pointer, absolute-paths-not-starting-with-crate, unstable-name-collisions, where-clauses-object-safety, proc-macro-derive-resolution-fallback, macro-expanded-macro-exports-accessed-by-absolute-paths, ill-formed-attribute-input, conflicting-repr-hints, ambiguous-associated-items, mutable-borrow-reservation-conflict, indirect-structural-match, pointer-structural-match, nontrivial-structural-match, soft-unstable, cenum-impl-drop-cast, const-evaluatable-unchecked, uninhabited-static, array-into-iter
          nonstandard-style  non-camel-case-types, non-snake-case, non-upper-case-globals
    rust-2018-compatibility  keyword-idents, anonymous-parameters, tyvar-behind-raw-pointer, absolute-paths-not-starting-with-crate
           rust-2018-idioms  bare-trait-objects, unused-extern-crates, ellipsis-inclusive-range-patterns, elided-lifetimes-in-paths, explicit-outlives-requirements
                    rustdoc  non-autolinks, broken-intra-doc-links, private-intra-doc-links, invalid-codeblock-attributes, missing-doc-code-examples, private-doc-tests, invalid-html-tags
                     unused  unused-imports, unused-variables, unused-assignments, dead-code, unused-mut, unreachable-code, unreachable-patterns, overlapping-patterns, unused-must-use, unused-unsafe, path-statements, unused-attributes, unused-macros, unused-allocation, unused-doc-comments, unused-extern-crates, unused-features, unused-labels, unused-parens, unused-braces, redundant-semicolons
```



