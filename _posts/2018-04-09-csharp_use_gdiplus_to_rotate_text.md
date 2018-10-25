---
layout: post
title: 'C# 使用 GDI+ 实现添加中心旋转(任意角度)的文字'
categories: 后台
# cover: '/assets/img/3.jpg'
tags: C#
excerpt: '本文介绍了C# 如何利用 GDI+ 来写上以文字内容为中心旋转的文字，同理也可扩展到如何画出以任意位置为中心旋转的图片。'
---

这篇文章是 `GDI+` 总结系列的第三篇，如果对 `GDI+` 的基础使用不熟悉的朋友可以先看第一篇文章[《C# 使用 GDI+ 画图》](https://dandelion-drq.github.io/2018/04/01/use-gdiplus-to-draw-image-in-csharp.html)。

---

# 需求

需求是要实现给图片添加任意角度旋转的文字，文字的旋转中心要是在文字区域中央，就像 `CSS` 的 `rotate` 函数一样的效果。如下：  
![](http://120.77.171.203:8080/images/blog-img/11.gif)


---

# 分析&思路

`Graphics` 类有个 `RotateTransform` 方法，可以传入任意角度的值来旋转画板。但是这个方法的旋转中心是画板的左上角，所以直接单单用这个方法不能满足我们的需求。此外，`Graphics`类还有个 `TranslateTransform` 方法可以改变坐标的原点，而且这个方法是沿着矩形的x,y轴平移的，即就算图片旋转了一定的角度后，再调用 `TranslateTransform` 方法，它还是沿着x,y轴平移。于是通过以下三个步骤即可实现图片中心旋转。
1. 把画板(Graphics对象)原点平移到矩形中心位置(x, y)
2. 在(x, y)位置绕原点旋转画板N度
3. 画板退回(-x, -y)的距离

还是看不懂的同学看下面的图应该就明白了  
![](http://120.77.171.203:8080/images/blog-img/12.png)


明白了原理，那不容易推断出，如果要旋转的中心不是图片中心而是文字中心，那步骤还是一样的，只是把(x, y)改为文字中心的坐标就好了。

除了上面说的方法，其实还有一个方法可以实现中心旋转，那就是使用 `Matrix` 类。`Matrix` 类的 `RotateAt` 方法可以指定矩阵旋转的中心位置。
```csharp
//
// 摘要:
//     沿 point 参数中指定的点并通过预先计算该旋转，来顺时针旋转此 System.Drawing.Drawing2D.Matrix。
//
// 参数:
//   angle:
//     旋转角度（范围）（单位：度）。
//
//   point:
//     一个 System.Drawing.PointF，表示旋转中心。
[TargetedPatchingOptOut("Performance critical to inline this type of method across NGen image boundaries")]
public void RotateAt(float angle, PointF point);
```
`Graphics` 类的 `Transform` 属性返回的就是 `Matrix` 对象，该属性可以 `get`、`set`。因此我们先获取原来的画板的矩阵，然后使用 `RotateAt` 方法旋转该矩阵，再把旋转后的矩阵赋值给画板就好了。

---

# 具体实现

- 添加任意角度文字方法
```csharp
/// <summary>
/// 图片添加任意角度文字(文字旋转是中心旋转，角度顺时针为正)
/// </summary>
/// <param name="imgPath">图片路径</param>
/// <param name="locationLeftTop">文字左上角定位(x1,y1)</param>
/// <param name="fontSize">字体大小，单位为像素</param>
/// <param name="text">文字内容</param>
/// <param name="angle">文字旋转角度</param>
/// <param name="fontName">字体名称</param>
/// <returns>添加文字后的Bitmap对象</returns>
public Bitmap AddText(string imgPath, string locationLeftTop, int fontSize, string text, int angle = 0, string fontName = "华文行楷")
{
    Image img = Image.FromFile(imgPath);
    int width = img.Width;
    int height = img.Height;
    Bitmap bmp = new Bitmap(width, height);
    Graphics graphics = Graphics.FromImage(bmp);
    // 画底图
    graphics.DrawImage(img, 0, 0, width, height);
    Font font = new Font(fontName, fontSize, GraphicsUnit.Pixel);
    SizeF sf = graphics.MeasureString(text, font); // 计算出来文字所占矩形区域
    // 左上角定位
    string[] location = locationLeftTop.Split(',');
    float x1 = float.Parse(location[0]);
    float y1 = float.Parse(location[1]);
    // 进行文字旋转的角度定位
    if (angle != 0)
    {
        #region 法一：TranslateTransform平移 + RotateTransform旋转
        /* 
            * 注意：
            * Graphics.RotateTransform的旋转是以Graphics对象的左上角为原点，旋转整个画板的。
            * 同时x，y坐标轴也会跟着旋转。即旋转后的x，y轴依然与矩形的边平行
            * 而Graphics.TranslateTransform方法，是沿着x，y轴平移的
            * 因此分三步可以实现中心旋转
            * 1.把画板(Graphics对象)平移到旋转中心
            * 2.旋转画板
            * 3.把画板平移退回相同的距离(此时的x，y轴仍然是与旋转后的矩形平行的)
            */
        //// 把画板的原点(默认是左上角)定位移到文字中心
        //graphics.TranslateTransform(x1 + sf.Width / 2, y1 + sf.Height / 2);
        //// 旋转画板
        //graphics.RotateTransform(angle);
        //// 回退画板x,y轴移动过的距离
        //graphics.TranslateTransform(-(x1 + sf.Width / 2), -(y1 + sf.Height / 2));
        #endregion
        #region 法二：矩阵旋转
        Matrix matrix = graphics.Transform;
        matrix.RotateAt(angle, new PointF(x1 + sf.Width / 2, y1 + sf.Height / 2));
        graphics.Transform = matrix;
        #endregion
    }
    // 写上自定义角度的文字
    graphics.DrawString(text, font, new SolidBrush(Color.Black), x1, y1);
    graphics.Dispose();
    img.Dispose();
    return bmp;
}
```
PS：这里简单解释一下为什么文字中心是 `(x1 + sf.Width / 2, y1 + sf.Height / 2)`，因为 `(x, y)` 是左上角，而 `sf.Width`、`sf.Height` 是文字矩形区域宽、高。如图：  
![](http://120.77.171.203:8080/images/blog-img/13.jpg)


- 测试调用
```csharp
private static void Main(string[] args)
{
    try
    {
        Console.WriteLine("Start drawing ...");
        DrawingEntity drawing = new DrawingEntity();
        System.Drawing.Bitmap bmp = drawing.AddText(@"D:\test\1.png", "176.94,150.48", 66, "写点啥好呢", 30);
        bmp.Save(@"D:\test\output.png");
        bmp.Dispose();
        Console.WriteLine("Done!");
    }
    catch (System.Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }
    finally
    {
        System.Console.WriteLine("\nPress any key to continue ...");
        System.Console.ReadKey();
    }
}
```

---

最终效果

- 没有旋转时  
![](http://120.77.171.203:8080/images/blog-img/14.png)


- 中心旋转30度  
![](http://120.77.171.203:8080/images/blog-img/15.png)


---

# 一个思考

讲完了大家来思考一个问题，如果我想做图片绕任意位置为中心进行旋转应该怎么做呢？相信看完了上面的代码大家应该都会了吧。

---

参考文章：

[C#中基于GDI+(Graphics)图像处理系列之任意角度旋转图像](https://blog.csdn.net/lhtzbj12/article/details/54099572)

[C#利用GDI+绘制旋转文字，矩形内可以根据布局方式排列文本](https://blog.csdn.net/alicehyxx/article/details/17009271)

---

系列其他文章：

[C# 使用 GDI+ 画图](https://dandelion-drq.github.io/2018/04/01/use-gdiplus-to-draw-image-in-csharp.html)

[C# 使用 GDI+ 给图片添加文字，并使文字自适应矩形区域](https://dandelion-drq.github.io/2018/04/07/csharp_use_gdiplus_to_add_text.html)
