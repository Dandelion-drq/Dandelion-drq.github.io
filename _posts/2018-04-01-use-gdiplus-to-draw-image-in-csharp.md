---
layout: post
title: 'C# 使用 GDI+ 画图'
categories: 后台
cover: 'http://120.77.171.203/images/blog-img/7.jpg'
tags: C#
excerpt: ' 本文介绍了 C# 使用 GDI+ 画图的一些基础知识，以及要注意的一些坑。'
---

最近做一个微信公众号服务，有一些简单的图片处理功能。主要就是用户在页面操作，前端做一些立刻显示的效果，然后提交保存时后端真正修改原图。   
我们的后端是 `ASP.NET`，也就是 `C#` 语言了，`C#` 本身处理图片还是比较方便的，使用 `GDI+` 就好，只需要添加 `System.Drawing` 引用，不需要任何第三方库。于是最近也用到一些比较常用的 `GDI+` 图片处理方法，就整理一下做个记录了。

这个题目大概会写几篇文章，第一篇先简单介绍一下 `GDI+` 的常用对象，以及一些使用时候的注意事项，后面会挑一些项目中做过的比较有用的处理过程来介绍一下。

废话不多说，开始进入正题。

---

# 需要用到的类

使用 `GDI+` 画图会用到的几个常用的类有：`Graphics`、`Bitmap`、`Image`。  
其中 `Graphics` 是画板。这个类包含了许多画图的方法，包括画图片(`DrawImage`)，画线(`DrawLine`)，画圆(`DrawEllipse、FillEllipse`)，写字(`DrawString`)等等。简单说使用这个类可以完成我们需要的大部分工作。 

生成一个 `Graphics` 对象需要用到 `Image` 或者 `Bitmap`。  
**PS：** `Winform` 下可以直接从窗体或控件的事件中引用 `Graphics` 对象。
比如：
```csharp
    private void Form1_Paint(object sender, PaintEventArgs e)
    {
        Graphics g = e.Graphics; // 创建画板，这里的画板是由Form提供的.
    }
```

不过本文讨论的是其他场景，比如 `ASP.NET MVC`，或单纯的控制台程序。这些时候是没有控件的，所以要用其他方法。  

我一般用以下方法：
```csharp
//
// 摘要:
//     从指定的 System.Drawing.Image 创建新的 System.Drawing.Graphics。
//
// 参数:
//   image:
//     从中创建新 System.Drawing.Graphics 的 System.Drawing.Image。
//
// 返回结果:
//     此方法为指定的 System.Drawing.Image 返回一个新的 System.Drawing.Graphics。
//
// 异常:
//   T:System.ArgumentNullException:
//     image 为 null。
//
//   T:System.Exception:
//     image 具有索引像素格式，或者格式未定义。
public static Graphics FromImage(Image image);
```
其中的参数可以传入 `Image` 或 `Bitmap`，因为 `Bitmap` 是继承自 `Image` 的。

---

# 如何创建画板

- 如果是要对原图进行处理，比如旋转图片，添加文字等，可以直接通过原图片获得画板对象。
```csharp
Image img = Image.FromFile(imgPath);
Graphics graphics = Graphics.FromImage(img);
```

- 如果是要画一个新的图，可以通过要保存的图片宽、高生成画板。
```csharp
Bitmap bmp = new Bitmap(width, height);
Graphics graph = Graphics.FromImage(bmp);
```
**PS：** `Graphics` 本身是没有提供构造函数来直接生成的。所以我们可以先创建一个需要保存图片大小的 `Bitmap` 位图对象，然后再获得画板对象。

---

# 如何保存画好的图片
通过调用 `img.Save(savePath)` 或者 `bmp.Save(savePath)` 即可保存对象。  
**PS：** `Bitmap` 的 `Save` 方法是直接继承自 `Image` 的。

---

# `GDI+` 的坐标系
`GDI+` 的坐标系是个二维坐标系，不过又有点不一样，它的原点是在左上角的。如下图：
![](https://images2018.cnblogs.com/blog/893839/201804/893839-20180401170914816-1168338857.png)

---

# 使用 `GDI+` 的一些注意事项

这里我忍不住要先吐槽一下，`GDI+` 的报错信息不太友好啊。经常只是返回一个“GDI+ 中发生一般性错误。”，不能快速地根据这个错误提示定位问题。比如说没有释放图片资源时想再次访问资源会报这个错误，想要保存图片的文件夹不存在时也是提示这个错误。看不出来区别……

**1. 保存到相同路径的文件时要先释放图片资源，否则会报错(GDI+中发生一般性错误)**
```csharp
Image img = Image.FromFile(imgPath);
Bitmap bmp = new Bitmap(img);
Graphics graphics = Graphics.FromImage(bmp);
... // 对图片进行一些处理
img.Dispose(); // 释放原图资源
bmp.Save(imgPath); // 保存到原图
graphics.Dispose(); // 图片处理过程完成，剩余资源全部释放
bmp.Dispose();
```

**2. 使用完的资源记得要释放。可以用 `try..catch..finally` 或者 `using` 的方式，这样即使遇到代码运行报错也能及时释放资源，更加保险。**

- `try..catch...finally`：把释放资源的代码写到 `finally` 代码段里。
```csharp
    Image img = Image.FromFile(imgPath);
    Bitmap bmp = new Bitmap(img);
    Graphics graphics = Graphics.FromImage(bmp);

    try
    {
        ...
    }
    catch (System.Exception ex)
    {
        throw ex;
    }
    finally
    {
        graphics.Dispose();
        bmp.Dispose();
        img.Dispose();
    }
```

- `using`：使用 `using` 语句创建的资源会在离开 `using` 代码段时自动释放该资源。
```csharp
    /// <summary>
    /// 缩放图像
    /// </summary>
    /// <param name="originalImagePath">原图路径</param>
    /// <param name="destWidth">目标图宽度</param>
    /// <param name="destHeight">目标图高度</param>
    /// <returns></returns>
    public Bitmap GetThumbnail(string originalImagePath, int destWidth, int destHeight)
    {
        using (Image imgSource = Image.FromFile(originalImagePath))
        {
            return GetThumbnail(imgSource, destWidth, destHeight);
        }
    }
```

**3. 要保存图片的文件夹一定要是已经存在的，否则会报错(GDI+中发生一般性错误)**

eg：假设图片要保存到 `D:\test\output.png`
```csharp
    string directory = @"D:\test\";
    string fileName = "output.png";

    // 检查文件夹是否存在，不存在则先创建
    if (!Directory.Exists(directory))
    {
        Directory.CreateDirectory(directory);
    }

    bmp.Save(directory + fileName);
```

---

系列其他文章：

[C# 使用 GDI+ 给图片添加文字，并使文字自适应矩形区域](https://dandelion-drq.github.io/2018/04/07/csharp_use_gdiplus_to_add_text.html)

[C# 使用 GDI+ 实现添加中心旋转(任意角度)的文字](https://dandelion-drq.github.io/2018/04/09/csharp_use_gdiplus_to_rotate_text.html)