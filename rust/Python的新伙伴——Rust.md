[[TOC]]

# Python的新伙伴——Rust

面对Python性能的瓶颈，我们往往会将耗时的部分使用C编写成dll后使用Python调用。幸运的是，这种方式有很好的性能的提升，可以满足需要。但不幸的是用C语言在写代码、调试的过程非常痛苦，动不动就会遇到内存错误。那么，有没有其它的方法，即能提高性能，又让减少程序编写的错误呢。Mazilla给了我们答案：Rust。

## Why Rust

Rust语言是一个极其讲究精致的系统级别语言，它瞄准的是取代C/C++。

Rust的中文意思是腐蚀，又拿取代C/C++为目标。那么它是怎么腐蚀C/C++的呢？C/C++从代码到可执行程序，中间要经过两个步骤：编译和链接。编译器将代码编译为.o文件，链接器将多个.o链接为可执行程度。而Rust只有编译器，没有链接器。因为Rust的编译器将代码编译为和C/C++一模一样的.o文件，然后，直接使用C/C++的链接器生成可执行程序。因此，Rust拥有着与C/C++不相上下的执行速度。

Rust管理内存的方法是世界上最先进的方法！C/C++本身没有内存管理，这需要依赖程序员的经验。而即使经验丰富的程序员也很难做到万无一失。而一般的带有内存管理的程序设计语言都采用GC（垃圾回收）的方法管理内存，这种方式本身就有不小的内存和时间开销。Rust采用了RAII(Resource Acquisition Is Initialization)实现资源自动释放。这种方法与GC最大的区别在于GC是工作在执行期，而RAII工作在编译期。这样不但能提供内存安全、供数据和资源的安全，还能做到几乎没有内存和时间的开销。

相比C/C++，Rust提供了更清晰、更简洁的语法，使开发者更好的掌握。

## 对Rust的态度

上面说了Rust的那么多优点，事实上，Rust也有着不小的缺点。它虽然比起C/C++的语法简单了，但要熟练使用Rust，依然有着陡峭的学习曲线。并且Rust现在的社区、第三方库还非常不成熟。  

目前网络上对Rust的态度处于两个极端，觉得它好的人认为Rust中的理念无一不是世界最先进的，随着WebAssembly的发展，未来Rust不但可以代替C/C++，还能代替JavaScript。觉得它不好的人认为Rust的学习曲线如此陡峭，始终会是一个被边缘化的语言。

对Rust，我是非常喜欢的，但面对Rust的诸多不成熟，我还是用观望态度期待Rust能够蓬勃发展。我写这篇文章，也不是决定了以后就选择用Rust写dll，只是学习一下如何使用Rust来解决这样的问题，通过学习来观望Rust的发展。

## 使用Rust开发的一般过程

Rust还有一大优点就是为开发者提供了一套完整的工具链。使用工具链，可以方便进行开发。这里只简单的介绍工具链中的cargo，cargo类似于Python中的pip，但远比pip强大。使用cargo可以创建工程、编译工程、运行工程、测试工程等。在编译时发现没有安装的库，cargo会自动进行安装，安装完后继续编译。

使用如下命令创建工程：

```
cargo init testlib --lib
```

其中init表示初始化工程，testlib为工程名，--lib意为工程的类型为动态库。创建完成后，生成了如下的目录结构：

```
testlib
├─ Cargo.toml       记录工程的作者、版本号、依赖库等的工程文件
└─ src              源代码目录
   └─ lib.rs        源代码文件
```

在lib.rs中，写入如下代码：
```rs
#[no_mangle]
pub extern fn add_int(a: i32, b: i32) -> i32
{
    println!("a = {}, b = {}", a, b);

    return a + b;
}

```

Rust的编译器在编译的过程中，会自动的对函数名进行变更处理，这对Rust本身没有什么影响，但对于dll中导出的函数，函数名是不能改变的。#[no_mangle]的意思就是告诉编译器，不要改变函数的名称。  

这个函数的作用是从Python接收两个32位的整数，打印这两个数的值，并返回它们相加后的结果。

在Cargo.toml的开头，添加如下三行：

```toml
[lib]
name = "testlib"
crate-type = ["cdylib"]
```

其中name为库的名称，crate-type为C动态库。

使用如下命令编译：

```
cargo build --release
```

使用release方式编译，编译得到的dll可以脱离Rust环境。

在Cargo.toml的同级创建test.py，在Python中通过ctypes调用。

```python
from ctypes import CDLL

dll = CDLL("target/release/testlib.dll") 
print(dll.add_int(1, 102))
```

执行结果如下：
```
a = 1, b = 102
103
```

## 各种参数的返回值类型的传递

ctypes提供了Python中数据类型和C的数据类型的转换，在此我们扩充一下对Rust的数据类型的转换关系表：

ctypes数据类型 | Rust数据类型 | C数据类型
:-- | :-- | :--
c_char | | char
c_short | | short
c_ushort | | unsigned short
c_int | | int
c_uint | | unsigned int
c_long | | long
c_ulong | | unsigned long
c_float | | float
c_double | | double
