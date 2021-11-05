## 动态类型

大多数类型都有一个在编译时已知的固定大小，并实现`Size` trait ，且只有在运行时才知道大小的类型称为动态类型(DST)

- 指向DST的指针类型实现了Size trait，但其大小是指向大小类型的指针的两倍。
  - 指向切片的指针还存储着切片元素数
  - trait 对象的指针还存储着指向vtable的指针
- DST可以作为类型参数提供，`?Size` ,默认情况下，任何类型参数都有大小限制。
- DST类型实现的trait，不同于`Self::?Sized`在trait定义中默认设置大小
- struct 可能包含一个DST类型作为最后一个字段，这也使得该struct变为了DST类型。

注：变量，函数参数，常量，static 必须是`Sized`

## Type Layout

### 原始数据布局

| 类型      | `size_of::<Type>()` |
| --------- | ------------------- |
| bool      | 1                   |
| u8/i8     | 1                   |
| u16/i16   | 2                   |
| u32/i32   | 4                   |
| u64/i64   | 8                   |
| u128/i128 | 16                  |
| f32       | 4                   |
| f64       | 8                   |
| char      | 4                   |

### 指针和引用布局

指针和引用具有相同的布局。指针或引用的可变性不会改变布局

指向`size`类型的指针大小和对齐方式与`usize`相同。

注：所有指向DST的指针当前的大小都是的大小的两倍，并且具有`usize`相同的大小对齐

### 数组布局

数组布局使得数组的第m隔元素从数组的开头偏移**m类型字节的大小**，`[T;m]` 的大小:`size_of::<T>()*n` 和T相同的对齐方式。

### slice layout

切片与其原数组具有相同的布局

注：这是关于原始的[T]类型，而不是指向切片的指针（&[T]、Box<[T]>等）。

### str layout

字符串片段是`UTF-8`表示的字符，其布局与[u8]类型的片段相同。

### Tuple Layout

元组对其布局没有任何保证

`()` 零尺寸类型，size为0，对齐方式为1

### trait 对象布局

trait对象的布局和trait对象的值相同。

注：这是关于原始trait对象类型，而不是指向trait对象的指针(&dyn trait 、Box<dyn trait>)

### closure layout

闭包没有布局

### Representition

自定义的复合类型(struct、enum、union)都有一个表现形式，用于指定类型的布局：

- Default 
- C
- transparent
- The primitive representation

类新的布局通过其对应的`repr`属性进行更改

```rust
#[repr(C)]
struct A{
	first:i32,
    second:u16,
    third:i64
}
```

对齐方式可以分别用*align*和*packed*进行设置。它们改变属性中指定的表示形式。如果未指定任何表示形式，则会更改默认的表示形式。

```rust
// C对齐方式 
#[repr(C,align(8))]
struct AlignA{
	first:i32,
    second:u16,
    third:i64
}


//默认表示方式，对齐方式为2
#[repr(packed(2))]
struct PackedA{
	first:i32,
    second:u16,
    third:i64
}
```

注意：由于表示是项的一个属性，因此表示不依赖于泛型参数。具有相同名称的任何两个类型具有相同的表示形式。例如，Foo<Bar>和Foo<Baz>具有相同的表示形式。

类型的表示可以更改字段之间的填充，但不会更改字段本身的布局。具有C表示形式且包含具有默认表示形式的结构内部的结构不会更改内部的布局。

### The Default Representation

没有repr属性的标称类型具有默认表示形式,此不能保证数据布局。

###  #[repr(C)] structs

> 结构体对齐方式是结构中最对齐的字段对齐方式，从当前偏移0字节开始。
>
> 对于结构中按声明顺序排列的每个字段，首先确定字段的大小和对齐方式。如果当前偏移量不是字段对齐方式的倍数，则将填充字节添加到当前偏移量，直到它是字段对齐方式的倍数。该字段的偏移量就是当前的偏移量。然后，将当前偏移量增加字段的大小。

### #[repr(C)] unions

使用`#[repr(C)]`声明的Union具有和目标平台的Clang中等效的C联合声明相同的大小和对齐方式。Union的大小将四舍五入到其对齐方式的所有字段的最大大小，并对齐其所有字段的最大对齐的对齐方式。

```rust
#[repr(C)]
union Union {
    f1: u16,
    f2: [u8; 4],
}

assert_eq!(std::mem::size_of::<Union>(), 4);  // From f2
assert_eq!(std::mem::align_of::<Union>(), 2); // From f1

#[repr(C)]
union SizeRoundedUp {
   a: u32,
   b: [u16; 3],
}

assert_eq!(std::mem::size_of::<SizeRoundedUp>(), 8);  // Size of 6 from b,
                                                      // rounded up to 8 from
                                                      // alignment of a.
assert_eq!(std::mem::align_of::<SizeRoundedUp>(), 4); // From a
```

### #[repr(C)] Field-Less enum

对于无字段枚举，C表示法的大小和对齐方式与目标平台的C ABI的默认枚举大小和对齐方式相同。

> 注：C语言中的枚举与具有此表示形式的Rust的无字段枚举之间存在关键区别。 C中的枚举主要是typedef加上一些命名常量；换句话说，枚举类型的对象可以保存任何整数值。例如，它经常用于C语言中的位标志。相反，Rust的无字段枚举只能合法地保存可区分的值，其他所有行为都是未定义的行为。因此，在FFI中使用无字段枚举来建模C枚举通常是错误的。

### #[repr(C)]  Enum With Fields

具有字段的`#[repr(C)]`枚举的表示形式是具有两个字段的`[repr(C)]` Struct，在C中也称为“标记联合”：

- `repr(C)` 删除所有字段的枚举版本(tag)
- 具有`repr(C)`结构的每个变量的字段的`repr(C)`union（“有效负载”）

> 由于repr（C）结构和联合的表示形式，如果变量具有单个字段，则直接将该字段放入联合或将其包装在结构中没有区别。



```rust

#![allow(unused)]
fn main() {
// This Enum has the same representation as ...
#[repr(C)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... this struct.
#[repr(C)]
struct MyEnumRepr {
    tag: MyEnumDiscriminant,
    payload: MyEnumFields,
}

// This is the discriminant enum.
#[repr(C)]
enum MyEnumDiscriminant { A, B, C, D }

// This is the variant union.
#[repr(C)]
union MyEnumFields {
    A: MyAFields,
    B: MyBFields,
    C: MyCFields,
    D: MyDFields,
}

#[repr(C)]
#[derive(Copy, Clone)]
struct MyAFields(u32);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyBFields(f32, u64);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyCFields { x: u32, y: u8 }

// This struct could be omitted (it is a zero-sized type), and it must be in
// C/C++ headers.
#[repr(C)]
#[derive(Copy, Clone)]
struct MyDFields;
}

```



注： 

- 基本表示形式是与基本整数类型具有相同名称的表示形式。即：u8，u16，u32，u64，u128，usize，i8，i16，i32，i64，i128和isize。原始表示形式只能应用于枚举，并且无论枚举有字段还是无字段都具有不同的行为。零变量枚举具有原始表示形式是一个错误。将两个原始表示形式组合在一起是一个错误。
- 对于无字段枚举，原始表示将大小和对齐方式设置为与相同名称的原始类型相同。
- 基本表示枚举的表示形式是具有字段的每个变量的`repr(C)`结构的`repr(C)`联合。联合中每个结构的第一个字段是枚举的原始表示形式，其中所有字段都已删除（tag），其余字段是该变体的字段。
- 对于具有字段的枚举，也可以将repr（C）和原始表示形式（例如repr（C，u8））组合在一起。这通过将区分枚举的表示形式更改为所选原语来修改repr（C）。

### alignment modifiers

`align` `packed` 可用于更改union的对齐方式。packed还可能改**变字段之间的填充**。

将对齐方式指定为整数参数，形式为`#[repr(align(x))]`或＃`#[repr(packed(x))]`。对齐值必须是从1到229的2的幂。

- 对于packed，如＃`#[repr(packed]`中未给出值，则该值为1。
- 对于pakced，如果指定的对齐方式大于没有指定packed类型的对齐方式，则对齐方式和布局不受影响。为了定位字段，每个字段的对齐方式是指定的对齐方式和字段类型的对齐方式中的较小者。

- 对于align 如果指定的对齐方式小于没有align修饰符的类型的对齐方式，则该对齐方式不受影响。
- align和packed修饰符不能应用于同一类型，并且packet类型不能传递包含另一个aligned类型。 align和packed只能应用于默认和C表示形式。
- align修饰符也可以应用于枚举。启用时，对枚举的对齐方式的影响与使用相同align修饰符将枚举包裹在新类型结构中时的效果相同。

> 取消引用未对齐的指针是未定义的行为，并且可以安全地创建指向打包字段的未对齐的指针。像在安全Rust中创建未定义行为的所有方法一样，这是一个错误。

### transparent repersentation

只能用于具有以下单个变量的结构或枚举：

- 大小非零的单个字段
- 大小为0且对齐方式为1的任意数量的字段

具有此表示形式的结构和枚举与单个非零大小字段具有相同的布局和ABI。

这与C表示形式不同，因为具有C表示形式的结构将始终具有C结构的ABI，例如，具有具有原始字段的透明表示形式的结构将具有原始字段的ABI。不能与其他任何表示形式一起使用。







