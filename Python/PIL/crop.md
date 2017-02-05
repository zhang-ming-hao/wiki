# 使用crop函数对图像进行裁切

crop的源码注释如下：
```python
def crop(self, box=None):
    """
    Returns a rectangular region from this image. The box is a
    4-tuple defining the left, upper, right, and lower pixel
    coordinate.

    This is a lazy operation.  Changes to the source image may or
    may not be reflected in the cropped image.  To break the
    connection, call the :py:meth:`~PIL.Image.Image.load` method on
    the cropped copy.

    :param box: The crop rectangle, as a (left, upper, right, lower)-tuple.
    :rtype: :py:class:`~PIL.Image.Image`
    :returns: An :py:class:`~PIL.Image.Image` object.
    """
```

该函数的参数是左、上、右、下四个像素值。另外该函数不会改变源图像，而是返回一个新的图像对象。