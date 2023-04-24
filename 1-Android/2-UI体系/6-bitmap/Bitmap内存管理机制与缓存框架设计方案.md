> version：2022/07/
>
> review：



目录

[TOC]



# 关键词



# 一、前置知识

不论什么样的库，或者项目，都是有不同的基础知识点组织起来的，因此先掌握基础是很有必要的。

基础知识这块可以不写。

# 一、概述



# 二、Bitmap内存优化

Bitmap内存优化思路：

1、像素优化

1. 透明度优化

   对于不需要透明度的图片。可以**去除透明图**。使用 Bitmap.Config.RGB_565 格式。

2. 缩放优化

   **配置inSampleSize**。inSampleSize的作用是设置对图片进行采样的参数，比如原始图片为128*128，inSampleSize为4，那么结果图片将是(128/4) * (128/4)，即宽高均是之前的1/4，像素是之前的1/16。

   inSampleSize的取值，可以根据Bitmap要展示到的View的宽高来计算。如，128*128的原始图片，要展示到

   64*64的ImageView上，inSampleSize就是128/64。

示例：

```java
public static Bitmap resizeBitmap(Resources resources, int id, int maxW, int maxH, boolean hasAlpha) {
    BitmapFactory.Options options = new BitmapFactory.Options();
    // 只解码出相关的参数
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(resources, id, options);
    int originalWidth = options.outWidth;
    int originalHeight = options.outHeight;
    // 设置缩放系数，只能取2的n次方
    options.inSampleSize = calculateInSampleSize(originalWidth, originalHeight, maxW, maxH);
    if (!hasAlpha) {
        options.inPreferredConfig = Bitmap.Config.RGB_565;
    }
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(resources, id, options);
}

    private static int calculateInSampleSize(int originalWidth, int originalHeight, int maxW, int maxH) {
        int inSampleSize = 1;
        if (originalWidth > maxW && originalHeight > maxH) {
            inSampleSize = 2;
            while (originalWidth / inSampleSize > maxW && originalHeight / inSampleSize > maxH) {
                inSampleSize *= 2;
            }
        }
        return inSampleSize;
    }
```



2、











































# 相关问题

<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



<font color='orange'>Q：</font>



# 总结

1、

## 【精益求精】我还能做（补充）些什么？

1、



# 脑图



# 参考

