---
layout: post
title: 'C# 使用 GDI+ 给图片添加文字，并使文字自适应矩形区域'
categories: 后台
tags: C#
excerpt: '本文介绍了 C# 如何使用 GDI+ 为图片添加文字，并且可以使文字字体大小自适应。'
---

这篇文章是 `GDI+` 总结系列的第二篇，如果对 `GDI+` 的基础使用不熟悉的朋友可以先看第一篇文章[《C# 使用 GDI+ 画图》](https://dandelion-drq.github.io/2018/04/01/use-gdiplus-to-draw-image-in-csharp.html)。

---

# 需求

需求是要做一个编辑文字的页面。用户在网页端写文字，文字区域是个矩形框，用户可以通过下方的拖动条调节文字大小。  
如下图：  
![](http://120.77.171.203:8080/images/blog-img/9.png)

提交数据的时候前端传文字区域的左上角和右下角定位给后台。因为前端的字体大小单位与后端没什么关系，所以不能直接传字体大小，也就是后端要根据矩形区域以及文字内容来自己推算用什么样的字体大小合适。

简单说就是知道文字的矩形区域，以及文字内容，要让文字内容根据矩形区域大小调整到适合的字体大小能比较合适地填满这个区域。

---

# 分析&思路

`Graphics` 类有个 `MeasureString` 方法，可以用来计算以当前字体写出来的文字会占据多少像素。  
如下：
```csharp
//
// 摘要:
//     测量用指定的 System.Drawing.Font 绘制的指定字符串。
//
// 参数:
//   text:
//     要测量的字符串。
//
//   font:
//     System.Drawing.Font，它定义字符串的文本格式。
//
// 返回结果:
//     此方法返回 System.Drawing.SizeF 结构，该结构表示 text 参数指定的、使用 font 参数绘制的字符串的大小，单位由 System.Drawing.Graphics.PageUnit
//     属性指定。
//
// 异常:
//   T:System.ArgumentException:
//     font 为 null。
public SizeF MeasureString(string text, Font font);
```

这个方法返回的 `SizeF` 包含 `Width` 和 `Height` 属性，读取这两个属性可以获取到文字内容所占的宽高(以像素为单位)。
```csharp
//
// 摘要:
//     获取或设置此 System.Drawing.SizeF 结构的水平分量。
//
// 返回结果:
//     此 System.Drawing.SizeF 结构的水平分量，通常以像素为单位进行度量。
public float Width { get; set; }

// 摘要:
//     获取或设置此 System.Drawing.SizeF 结构的垂直分量。
//
// 返回结果:
//     此 System.Drawing.SizeF 结构的垂直分量，通常以像素为单位进行度量。
public float Height { get; set; }
```

于是我们可以先根据前端传过来的文字左上角与右下角定位，算出文字的矩形区域，然后估计一个字体大小，再用 `MeasureString` 方法计算出估算的文字所占区域，比较和实际的文字区域大小，大了则缩小字体，小了则增大字体。这样即可大约找出合适的文字大小。

---

# 具体实现

- 添加文字方法
```csharp
    /// <summary>
    /// 图片添加文字，文字大小自适应
    /// </summary>
    /// <param name="imgPath">图片路径</param>
    /// <param name="locationLeftTop">左上角定位(x1,y1)</param>
    /// <param name="locationRightBottom">右下角定位(x2,y2)</param>
    /// <param name="text">文字内容</param>
    /// <param name="fontName">字体名称</param>
    /// <returns>添加文字后的Bitmap对象</returns>
    public static Bitmap AddText(string imgPath, string locationLeftTop, string locationRightBottom, string text, string fontName = "华文行楷")
    {
        Image img = Image.FromFile(imgPath);

        int width = img.Width;
        int height = img.Height;
        Bitmap bmp = new Bitmap(width, height);
        Graphics graph = Graphics.FromImage(bmp);

        // 计算文字区域
        // 左上角
        string[] location = locationLeftTop.Split(',');
        float x1 = float.Parse(location[0]);
        float y1 = float.Parse(location[1]);
        // 右下角
        location = locationRightBottom.Split(',');
        float x2 = float.Parse(location[0]);
        float y2 = float.Parse(location[1]);
        // 区域宽高
        float fontWidth = x2 - x1;
        float fontHeight = y2 - y1;

        float fontSize = fontHeight;  // 初次估计先用文字区域高度作为文字字体大小，后面再做调整，单位为px

        Font font = new Font(fontName, fontSize, GraphicsUnit.Pixel);
        SizeF sf = graph.MeasureString(text, font);

        int times = 0;

        // 调整字体大小以适应文字区域
        if (sf.Width > fontWidth)
        {
            while (sf.Width > fontWidth)
            {
                fontSize -= 0.1f;
                font = new Font(fontName, fontSize, GraphicsUnit.Pixel);
                sf = graph.MeasureString(text, font);

                times++;
            }

            Console.WriteLine("一开始估计大了，最终字体大小为{0}，循环了{1}次", font.ToString(), times);
        }
        else if (sf.Width < fontWidth)
        {
            while (sf.Width < fontWidth)
            {
                fontSize += 0.1f;
                font = new Font(fontName, fontSize, GraphicsUnit.Pixel);
                sf = graph.MeasureString(text, font);

                times++;
            }

            Console.WriteLine("一开始估计小了，最终字体大小为{0}，循环了{1}次", font.ToString(), times);
        }

        // 最终的得出的字体所占区域一般不会刚好等于实际区域
        // 所以根据两个区域的相差之处再把文字开始位置(左上角定位)稍微调整一下
        x1 += (fontWidth - sf.Width) / 2;
        y1 += (fontHeight - sf.Height) / 2;

        graph.DrawImage(img, 0, 0, width, height);
        graph.DrawString(text, font, new SolidBrush(Color.Black), x1, y1);

        graph.Dispose();
        img.Dispose();

        return bmp;
    }
```

- 测试调用
```csharp
    private static void Main(string[] args)
    {
        try
        {
            DrawingEntity drawing = new DrawingEntity();

            Console.WriteLine("Start drawing ...");
            System.Drawing.Bitmap bmp = drawing.AddText(@"D:\test\39585148.png", "177.75,63.84", "674.73, 141.6", "大海啊，全是浪");
            bmp.Save(@"D:\test\output.png");
            bmp.Dispose();
            Console.WriteLine("Done!");
        }
        catch (System.Exception ex)
        {
            Console.WriteLine("出错了！！\n" + ex.ToString());
        }
        finally
        {
            System.Console.WriteLine("\nPress any key to continue ...");
            System.Console.ReadKey();
        }
    }
```

---

最终效果：

![](http://120.77.171.203:8080/images/blog-img/10.png)

---

系列其他文章：

[C# 使用 GDI+ 画图](https://dandelion-drq.github.io/2018/04/01/use-gdiplus-to-draw-image-in-csharp.html)

[C# 使用 GDI+ 实现添加中心旋转(任意角度)的文字](https://dandelion-drq.github.io/2018/04/09/csharp_use_gdiplus_to_rotate_text.html)