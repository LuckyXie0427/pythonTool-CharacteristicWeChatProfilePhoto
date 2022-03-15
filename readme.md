国庆节终于来了，最近有个五星红旗半透明渐变头像很火，在抖音里、微信群里都流行起来了。

那么，作为`python`玩家，是不是也要共享一下自己的力量，用**python快速制作这样的头像**呢！？

来吧，展示！

![](E:/%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91/others/python%E5%B0%8F%E5%B7%A5%E5%85%B7/%E7%94%A8python%E5%88%B6%E4%BD%9C%E4%BA%94%E6%98%9F%E7%BA%A2%E6%97%97%E5%9B%BD%E5%BA%86%E5%A4%B4%E5%83%8F/%E4%BA%94%E6%98%9F%E7%BA%A2%E6%97%97%E5%8D%8A%E9%80%8F%E6%98%8E%E6%B8%90%E5%8F%98%E5%A4%B4%E5%83%8F.png)

**目录：**

- 1、原理简介

<iframe src="//player.bilibili.com/player.html?aid=809858353&bvid=BV1834y1t7en&cid=549630250&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

- 2、实现步骤

- - 2.1. 读取图片
  - 2.2. 截取区域
  - 2.3. 设置透明渐变
  - 2.4. 粘贴到头像并保存

- 3、完整代码



#### 1、原理简介

我们看到这样的头像，大致是红旗透明度渐变然后覆盖在自己头像上即可。

那么，我的思路大致是先通过红旗图片获取和自己头像尺寸一样的区域，然后将这部分区域从左到右进行透明度渐变增长，然后将这张图片和头像进行融合，最终保存即可。

基于这个思路，结合之前学过的`PIL`库，我们大致可以将实现步骤拆分为如下几步：

- **读取国旗和头像照片** **`open`**
- **截取国旗部分区域** **`crop`**
- **从左到右透明度渐变** **`putpixel`**
- **将区域粘贴到头像** **`paste`+`resize`**
- **保存新头像** **`save`**

既然明确了实现步骤，我们就开搞吧！



#### 2、实现步骤

大家记得事先准备国旗和自己头像照片到本地哦

##### **2.1. 读取图片**

```
from PIL import Image

guoqi = Image.open('五星红旗.png').convert("RGBA")
touxiang = Image.open('头像.jpg').convert("RGBA")
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/vQr6oPKZqgvXpv4XvHP4ibWhIoJ1I7mCsTJ2zexfTKEoHJCnGlHkvGMQbP40hMIibxLnMLaqAlkTjpzTcByHhe6A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)																											五星红旗

<img src="E:/%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91/others/python%E5%B0%8F%E5%B7%A5%E5%85%B7/%E7%94%A8python%E5%88%B6%E4%BD%9C%E4%BA%94%E6%98%9F%E7%BA%A2%E6%97%97%E5%9B%BD%E5%BA%86%E5%A4%B4%E5%83%8F/%E5%A4%B4%E5%83%8F.jpg" style="zoom:200%;" />

​																													头像

##### **2.2. 截取区域**

由于这里我的头像是正方形，为了方便在粘贴透明渐变国旗时更方便，需要截取正方形区域。

```
# 获取国旗的尺寸
x,y = guoqi.size
# 根据需求，设置左上角坐标和右下角坐标（截取的是正方形）
quyu = guoqi.crop((50,20, y+62,y-100))
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/vQr6oPKZqgvXpv4XvHP4ibWhIoJ1I7mCsP4gucavloSOuoYasnb88PibacicRaelgWSf6UHLupibLdEqodjggJ7KCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)																									 五星红旗(正方形)

##### **2.3. 设置透明渐变**

在`PIL`库中，`getpixel((i, j))`表示获取`(i,j)`像素点的颜色值`color`，同样我们可以通过`putpixel((i, j), color)`来对`(i,j)`像素点设置颜色。

而对应`color`来说，是包含四个参数的元组`(R,G,B,alpha)`，分别是`RGB`值和`透明度`，其中**透明度255表示不透明，0表示100%透明**。

了解以上这些知识，我们就可以开始进行透明度渐变的操作了。

本例最简单满足需求的就是透明渐变从左到右透明度依次变高（参数值变小），考虑到从255变为0 且只能是整数，这里由于微信头像是`900*900`，所以我考虑的是**每3个像素进行一次透明度渐变**，当超过255之后则透明度为100%也就是对应参数为0。

```
# 获取头像的尺寸
w,h = touxiang.size
# 将区域尺寸重置为头像的尺寸
quyu = quyu.resize((w,h))
# 透明渐变设置
for i in range(w):
    for j in range(h):
        color = quyu.getpixel((i, j))
        alpha = 255-i//3
        if alpha < 0:
            alpha=0
        color = color[:-1] + (alpha, )
        quyu.putpixel((i, j), color)
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/vQr6oPKZqgvXpv4XvHP4ibWhIoJ1I7mCsUtP0rxTBgfL7su70QUgFibKdGHNhI3960KOnWWK8kxTte0FBBdJvLnw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​																										透明渐变

##### **2.4. 粘贴到头像并保存**

需要注意粘贴的时候要保留透明背景，否则就不好玩了，等于直接全覆盖

```
touxiang.paste(quyu,(0,0),quyu)
touxiang.save('五星红旗半透明渐变头像.png')
```

<img src="E:/%E5%90%8E%E5%8F%B0%E5%BC%80%E5%8F%91/others/python%E5%B0%8F%E5%B7%A5%E5%85%B7/%E7%94%A8python%E5%88%B6%E4%BD%9C%E4%BA%94%E6%98%9F%E7%BA%A2%E6%97%97%E5%9B%BD%E5%BA%86%E5%A4%B4%E5%83%8F/%E4%BA%94%E6%98%9F%E7%BA%A2%E6%97%97%E5%8D%8A%E9%80%8F%E6%98%8E%E6%B8%90%E5%8F%98%E5%A4%B4%E5%83%8F.png" style="zoom:200%;" />

​																									五星红旗国庆头像

#### 3、完整代码

```python
from PIL import Image

# 读取图片
guoqi = Image.open('五星红旗.png').convert("RGBA")
touxiang = Image.open('头像.jpg').convert("RGBA")

# 获取国旗的尺寸
x,y = guoqi.size
# 根据需求，设置左上角坐标和右下角坐标（截取的是正方形）
quyu = guoqi.crop((262,100, y+62,y-100))

# 获取头像的尺寸
w,h = touxiang.size
# 将区域尺寸重置为头像的尺寸
quyu = quyu.resize((w,h))
# 透明渐变设置
for i in range(w):
    for j in range(h):
        color = quyu.getpixel((i, j))
        alpha = 255-i//3
        if alpha < 0:
            alpha=0
        color = color[:-1] + (alpha, )
        quyu.putpixel((i, j), color)

# 粘贴到头像并保存 
touxiang.paste(quyu,(0,0),quyu)
touxiang.save('五星红旗半透明渐变头像.png')
```

