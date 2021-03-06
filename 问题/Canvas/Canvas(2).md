# Canvas基础二
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

上一次我们了解了一些Canvas的基础知识，这次依旧是Canvas相关的一些内容。

相比于上一次绘制简单的图形，这次的内容会稍微有趣一点点，相信大家看完之后会对Canvas更加了解,能够制作一些<b>更(geng)加(neng)炫(zhuang)酷(B)</b>的东西出来。

## 一.Canvas常用操作
为了方便，我把上次的表格搬过来了。

操作类型 | 相关API | 备注
--- | --- | ---
绘制颜色 | drawColor, drawRGB, drawARGB | 使用单一颜色填充整个画布<br/> 相关 : [安卓自定义View基础-颜色](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E9%A2%9C%E8%89%B2/%E9%A2%9C%E8%89%B2.md)
绘制基本形状 | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧
绘制图片 | drawBitmap, drawPicture | 绘制位图和图片
绘制文本 | drawText,    drawPosText, drawTextOnPath | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字
绘制路径 | drawPath | 绘制路径，绘制贝塞尔曲线时也需要用到该函数
顶点操作 | drawVertices, drawBitmapMesh | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用
画布剪裁 | clipPath,    clipRect | 设置画布的显示区域
画布快照 | save, restore, saveLayerXxx, restoreToCount, getSaveCount | 依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 会滚到指定状态、 获取保存次数
画布变换 | translate, scale, rotate, skew | 依次为 位移、缩放、 旋转、倾斜<br/> 相关： [坐标系](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E5%9D%90%E6%A0%87%E7%B3%BB/%E5%9D%90%E6%A0%87%E7%B3%BB.md)    [角度与弧度](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E8%A7%92%E5%BA%A6%E4%B8%8E%E5%BC%A7%E5%BA%A6/%E8%A7%92%E5%BA%A6%E4%B8%8E%E5%BC%A7%E5%BA%A6.md)
Matrix(矩阵) | getMatrix, setMatrix, concat | 实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。

******
## 二.Canvas基本操作
上次呢我们了解了绘制颜色和绘制基本形状，这次会了解画布变换相关操作。

*****
### 1.画布操作
#### 为什么要有画布操作？
画布操作可以帮助我们用更加容易理解的方式制作图形，如果你之前看过绘制太极和Canvas(1)最后的实例，你就会发现其中就有对translate的运用。

translate 是干什么用的呢？

上面表格中写的是"位移"，但"位移"的词义很是模糊，到底位移的是什么？那换种说法，translate是坐标系的移动，为图形绘制选择一个合适的坐标系，看下图：

<i>PS:请不要在意坐标系数值和图像的倾斜，制作软件出了一点问题，如果你有比较好的制作数学图形的软件可以推荐一下。</i>

![](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/translate.png)

不同的坐标系对于描述一个图形的难度实际上是不同的，就拿上图来说，假设蓝色矩形为屏幕，左侧状态为默认的坐标系，右侧为将坐标原点移动到屏幕中心的坐标系，要在屏幕中心绘制一个圆形，在左侧的坐标系中，你需要计算出圆心的位置和半径，而右侧只需知道半径即可。

<b>合理的使用画布操作可以帮助你用更容易理解的方式创作你想要的效果，这也是画布操作存在的原因。</b>

下面对几种画布操作详细讲解。

*****
#### ⑴位移(translate)
 <b>请注意，位移是基于当前位置移动，而不是每次基于屏幕左上角的(0,0)点移动</b>，如下：
``` java
        // 省略了创建画笔的代码
        
        // 在坐标原点绘制一个黑色圆形
        mPaint.setColor(Color.BLACK);
        canvas.translate(200,200);
        canvas.drawCircle(0,0,100,mPaint);

        // 在坐标原点绘制一个蓝色圆形
        mPaint.setColor(Color.BLUE);
        canvas.translate(200,200);
        canvas.drawCircle(0,0,100,mPaint);
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/translate.jpg" width = "270" height = "480" alt="title" align=center />  

我们首先将坐标系移动一段距离绘制一个圆形，之后再移动一段距离绘制一个圆形，<b>两次移动是叠加的</b>。

*****
#### ⑵缩放(scale)
缩放提供了两个方法，如下：
``` java
 public void scale (float sx, float sy)

 public final void scale (float sx, float sy, float px, float py)
```
这两个方法中前两个参数是相同的分别为x轴和y轴的缩放比例(1.0代表不变，大于1代表放大，小于1代表缩小)，而第二种方法比前一种多了两个参数。

多出来的两个参数是干什么的？ 当然是控制缩放中心位置的。

如果在缩放时稍微注意一下就会发现<b>缩放的中心默认为坐标原点</b>，如下：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.scale(0.5f,0.5f);                // 画布缩放

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
(为了更加直观，我添加了一个坐标系，可以比较明显的看出，缩放中心就是坐标原点)

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/scale1.jpg" width = "270" height = "480" alt="title" align=center />  

接下来我们使用第二种方法让缩放中心位置稍微改变一下，如下：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.scale(0.5f,0.5f,200,0);          // 画布缩放  <-- 缩放中心向右偏移了200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
(图中用箭头指示的就是缩放中心。)

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/scale2.jpg" width = "270" height = "480" alt="title" align=center />  

<b>PS:和位移(translate)一样，缩放也是可以叠加的。</b>
``` java
   canvas.scale(0.5f,0.5f);
   canvas.scale(0.5f,0.1f);
```
调用两次缩放则 x轴实际缩放为0.5x0.5=0.25 y轴实际缩放为0.5x0.1=0.05

下面我们利用这一特性制作一个有趣的图形。
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(-400,-400,400,400);   // 矩形区域

        for (int i=0; i<=20; i++)
        {
            canvas.scale(0.9f,0.9f);
            canvas.drawRect(rect,mPaint);
        }
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/scale3.jpg" width = "270" height = "480" alt="title" align=center />  

*****
#### ⑶旋转(rotate)
旋转提供了两种方法：
``` java
  public void rotate (float degrees)
  
  public final void rotate (float degrees, float px, float py)
```
和缩放一样，第二种方法多出来的两个参数依旧是控制旋转中心点的。

默认的旋转中心依旧是坐标原点：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.rotate(180);                     // 旋转180度 <-- 默认旋转中心为原点

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/rotate1.jpg" width = "270" height = "480" alt="title" align=center />  

改变旋转中心位置：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.rotate(180,200,0);               // 旋转180度 <-- 旋转中心向右偏移200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/rotate2.jpg" width = "270" height = "480" alt="title" align=center />  

<b>好吧，旋转也是可叠加的</b>
``` java
     canvas.rotate(180);
     canvas.rotate(20);
```
调用两次旋转，则实际的旋转角度为180+20=200度。

为了演示这一个效果，我做了一个不明觉厉的东西：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        canvas.drawCircle(0,0,400,mPaint);          // 绘制两个圆形
        canvas.drawCircle(0,0,380,mPaint);

        for (int i=0; i<=360; i+=10){               // 绘制圆形之间的连接线
            canvas.drawLine(0,380,0,400,mPaint);
            canvas.rotate(10);
        }
```
<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/rotate3.jpg" width = "270" height = "480" alt="title" align=center />  

*****
#### ⑷倾斜(skew)
skew这里翻译为倾斜，有的地方也叫错切。

倾斜只提供了一种方法：
``` java
  public void skew (float sx, float sy)
```
<b>参数含义：<br/>
float sx:将画布在x方向上倾斜相应的角度，sx倾斜角度的tan值，<br/>
float sy:将画布在y轴方向上倾斜相应的角度，sy为倾斜角度的tan值.</b>

示例：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,0,200,200);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.skew(1,0);                       // 在x轴倾斜45度 <-- tan45 = 1

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/skew1.jpg" width = "270" height = "480" alt="title" align=center />  

<b>如你所想，倾斜也是可叠加的，不过请注意，调用次序不同绘制结果也会不同</b>
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,0,200,200);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.skew(1,0);                       // 在x轴倾斜45度 <-- tan45 = 1
        canvas.skew(0,1);                       // 在y轴倾斜45度 <-- tan45 = 1

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/skew2.jpg" width = "270" height = "480" alt="title" align=center />  

*****
#### ⑸快照(save)和回滚(restore)

<b>
Q: 为什存在快照与回滚<br/>
A：画布的操作是不可逆的，而且很多画布操作会影响后续的步骤，例如第一个例子，两个圆形都是在坐标原点绘制的，而因为坐标系的移动绘制出来的实际位置不同。所以会对画布的一些状态进行保存和回滚。
</b>

<b>与之相关的API:</b>

相关API | 简介
--- | ---
save | 把当前的画布的状态进行保存，然后放入特定的栈中
saveLayerXxx | 新建一个图层，并放入特定的栈中
restore | 把栈中最顶层的画布状态取出来，并按照这个状态恢复当前的画布
restoreToCount| 弹出指定位置及其以上所有的状态，并按照指定位置的状态进行恢复
getSaveCount | 获取栈中内容的数量(即保存次数)

下面对其中的一些概念和方法进行分析：

##### 状态栈：
其实这个栈我也不知道叫什么名字，暂时叫做状态栈吧，它看起来像下面这样：

![](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/stack1.png)

这个栈可以存储画布状态和图层状态。

<b>Q：什么是画布和图层？<br/>
A：实际上我们看到的画布是由多个图层构成的，如下图(图片来自网络)：<br/>
![](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/layer.png)<br/>
实际上我们之前讲解的绘制操作和画布操作都是在默认图层上进行的。<br/>
在通常情况下，使用默认图层就可满足需求，但是如果需要绘制比较复杂的内容，如地图(地图可以有多个地图层叠加而成，比如：政区层，道路层，兴趣点层)等，则分图层绘制比较好一些。<br/>
你可以把这些图层看做是一层一层的玻璃板，你在每层的玻璃板上绘制内容，然后把这些玻璃板叠在一起看就是最终效果。
</b>

##### SaveFlags

数据类型 | 名称 | 简介
--- | --- | ---
int |	ALL_SAVE_FLAG	              | 默认，保存全部状态
int	| CLIP_SAVE_FLAG	             | 保存剪辑区
int	| CLIP_TO_LAYER_SAVE_FLAG	    | 剪裁区作为图层保存
int	| FULL_COLOR_LAYER_SAVE_FLAG	 | 保存图层的全部色彩通道
int	| HAS_ALPHA_LAYER_SAVE_FLAG	  | 保存图层的alpha(不透明度)通道
int	| MATRIX_SAVE_FLAG	           | 保存Matrix信息(translate, rotate, scale, skew)

##### save
save 有两种方法：
``` java
  // 保存全部状态
  public int save ()
  
  // 根据saveFlags参数保存一部分状态
  public int save (int saveFlags)
```
可以看到第二种方法比第一种多了一个saveFlags参数，使用这个参数可以只保存一部分状态，更加灵活，这个saveFlags参数具体可参考上面表格中的内容。

每调用一次save方法，都会在栈顶添加一条状态信息，以上面状态栈图片为例，再调用一次save则会在第5次上面载添加一条状态。

#### saveLayerXxx
saveLayerXxx有比较多的方法：
``` java
// 无图层alpha(不透明度)通道
public int saveLayer (RectF bounds, Paint paint)
public int saveLayer (RectF bounds, Paint paint, int saveFlags)
public int saveLayer (float left, float top, float right, float bottom, Paint paint)
public int saveLayer (float left, float top, float right, float bottom, Paint paint, int saveFlags)

// 有图层alpha(不透明度)通道
public int saveLayerAlpha (RectF bounds, int alpha)
public int saveLayerAlpha (RectF bounds, int alpha, int saveFlags)
public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha)
public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha, int saveFlags)
```
<b>注意：saveLayerXxx方法会让你花费更多的时间去渲染图像(图层多了相互之间叠加会导致计算量成倍增长)，使用前请谨慎，如果可能，尽量避免使用。</b>

使用saveLayerXxx方法，也会将图层状态也放入状态栈中，同样使用restore方法进行恢复。

这个暂时不过多讲述，如果以后用到详细讲解。(因为这里面东西也有不少啊QAQ)

##### restore
状态回滚，就是从栈顶取出一个状态然后根据内容进行恢复。

同样以上面状态栈图片为例，调用一次restore方法则将状态栈中第5次取出，根据里面保存的状态进行状态恢复。

##### restoreToCount
弹出指定位置以及以上所有状态，并根据指定位置状态进行恢复。

以上面状态栈图片为例，如果调用restoreToCount(2) 则会弹出 2 3 4 5 的状态，并根据第2次保存的状态进行恢复。

##### getSaveCount
获取保存的次数，即状态栈中保存状态的数量，以上面状态栈图片为例，使用该函数的返回值为5。

不过请注意，该函数的最小返回值为1，即使弹出了所有的状态，返回值依旧为1，代表默认状态。

##### 常用格式
虽然关于状态的保存和回滚啰嗦了不少，不过大多数情况下只需要记住下面的步骤就可以了：
``` java
   save();      //保存状态
   ...          //具体操作
   restore();   //回滚到之前的状态
```
这种方式也是最简单和最容易理解的使用方法。



******
## 三.总结

如本文一开始所说，合理的使用画布操作可以帮助你用更容易理解的方式创作你想要的效果。

其实之前准备在画布操作后再讲一些图片 文字 路径相关内容的， 但是没想到画布操作居然有这么多东西，这还是在没讲解Matrix的情况下。

(,,• ₃ •,,)

这次就先讲这么多吧。

<b>PS: 由于本人英文水平有限，某些地方可能存在误解或词语翻译不准确，如果你对此有疑问可以提交Issues进行反馈。</b>

******
## 四.参考资料

[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)<br/>
[canvas变换与操作](http://blog.csdn.net/harvic880925/article/details/39080931)<br/>
[Canvas之translate、scale、rotate、skew方法讲解](http://blog.csdn.net/tianjian4592/article/details/45234419)<br/>
[Canvas的save(),saveLayer()和restore()浅谈](http://www.cnblogs.com/liangstudyhome/p/4143498.html)<br/>
[Graphics->Layers](http://www.programgo.com/article/72302404062/;jsessionid=8E62016408BFFB21D46F9C878A49D8EE)<br/>
[AndroidのCanvasを使いこなす](http://tech.recruit-mp.co.jp/mobile/remember_canvas1/)<br/>
[]()<br/>
[]()<br/>

