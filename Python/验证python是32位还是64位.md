# 验证Python是32位还是64位

过去，我都是通过如下代码验证python是否为64位：

```python
import sys
print sys.maxint
```
此方法在linux/unix上屡试不爽，但是当我在windows上安装了64位的python后，发现即便是64位的python，sys.maxint也是2147483647，即2**31 - 1。  
后来在网上查询才明白，sys.maxint并不能表示python是32位还是64位，它是由当前编译环境的"long"类型所决定的。而在64位windows系统下, C的sizeof(long)通常也是4，所以sys.maxint与32位下相同。  
那么，如何验证python倒底是32位还是64位呢？

+ 使用platform
    ```python
    import platform
    print platform.architecture()
    ```
  如果输出('64bit', 'WindowsPE')，说明是64位的
+ 使用struct
    ```python
    import struct
    struct.calcsize("P")
    ```
  struct.calcsize用于计算格式字符串所对应的结果长度。如果输出结果为4，则为32位，如果输出结果为8，说明是64位。
    