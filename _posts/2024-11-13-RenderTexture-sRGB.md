---
layout: post
title: Unity中RenderTexture的sRGB问题
date: 2024-11-13 13:32:13
description: 如果使用Unity的RenderTexture来显示某个相机的内容，会发现相机的颜色和RenderTexture的颜色不一致。这是因为RenderTexture默认不打开sRGB，导致其内部存储的颜色其实是线性的。
tags: Unity, Rendering
categories: Computer Graphics
---

在Unity里中，如果使用编辑器(Editor)创建的RenderTexture来渲染相机的某个场景的图片，会发现渲染的场景物体下的真实颜色如果是灰色，在RenderTexture都会是更深的颜色，比如黑色。而从这个renderTexture上所导出的图像，比如附着在某个Texture2D上，后导出的PNG/JPG/bytes都会有更深的颜色。

项目里的内容不太方便截图，这里选用了一张同样总结该问题的其他post的图片。

![](/assets/img/post/24-11-13-render-texture-srgb/3.png)

其原因是：在编辑器下的RenderTexture默认是不打开sRGB的，所以其内部存储的颜色是线性的。

在这里最简单的解决办法是，在使用编辑器(Editor)创建RenderTexture时，点击有inspector右上角的三个点，把normal模式切换为debug模式。

![](/assets/img/post/24-11-13-render-texture-srgb/1.png)

然后在RenderTexture的inspector中，勾选sRGB。
![](/assets/img/post/24-11-13-render-texture-srgb/2.png)

当然，另一种办法是不要在编辑器(Editor)里创建RenderTexture，而是在代码中创建RenderTexture，然后在代码中设置RenderTexture的sRGB属性。

```csharp
RenderTexture rt = new RenderTexture(width, height, depth, RenderTextureFormat.ARGB32, RenderTextureReadWrite.sRGB);
```

这样就可以保证RenderTexture的颜色和Unity内部相机看到的颜色一致了！

参考资料：\
[1]: https://qiita.com/120byte/items/44df34fd30057f9642b4 \
[2]: https://discussions.unity.com/t/writting-to-rendertexture-comes-out-darker/637617 \
[3]:https://blog.csdn.net/lrh3025/article/details/102723090