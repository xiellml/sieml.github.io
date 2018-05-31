---
title: Custom View Components
date: 2018-03-28 11:28:45
tags: views
---

自定义View为什么这么难？他到底难在哪里，尤其是设计到动效和事件交互的时候。

那么，我们需要不断实践练习和补充完整的知识才行！这一篇权当入门。



我们从View 这个类本身入手：



#### 一. View的构造函数和常见函数

Constructor to use when creating a view from code.

- View(Context context)

Constructor that is called when inflating a view from XML.

- View(Context context, @Nullable AttributeSet attrs)
- View(Context context, @Nullable AttributeSet attrs, int defStyleAttr)
- View(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes)

后面三个方法的逻辑都是同一个，最后系统都会调用最后一个函数（因为其通过this(…)指向了4参函数）.
所以：

1. 如果不对二参数构造方法使用this(x,x,R.attr.CustomizeStyleAttr)进行传参，系统会将第三个参数设为0;
2. 如果不对三参数构造方法使用this(x,x,x,R.styl.CustomizeStyleRes)进行传参,系统会将第四个参数设为0。

四参构造如下：

```java
View(x,x,x,x){
    this(context);//首先也需要调用代码形式的构造方法
    final TypedArray a = context.obtainStyledAttributes(
                attrs, com.android.internal.R.styleable.View, defStyleAttr, defStyleRes);

        if (mDebugViewAttributes) {
            saveAttributeData(attrs, a);
    }
    //然后解析处TypeArray里面的参数值
    //其他...
}
```

总结: 解析属性的优先级： xml > style > defStyleAttr > defStyleRes > theme。
也就是主题theme作为当前Activity通用系的属性可以单独设置某个样式属性item，但是它只有在defStyleAttr和defStyleRes都为0的情况下才会使用自己的样式属性值。
**所以我们不用管defStyleAttr和defStyleRes让他采用系统属性（在主题theme更改系统属性值不推荐，除非是为了统一风格，针对单独的某个自定义View做更改会影响其他视图），可以采用自定义属性做到更改想要的效果而不影响其他的View（这个时候需要自己在调用super(…)之后将设定的属性值解析出来然后通过调用属性set函数设置到对应的代码属性之上）。**

- onFinishInflate():加载完XML组件后回调
- onSizeChanged():组件大小改变时回调
- onMeasure():回调该方法来进行测量（在该方法中实现对wrap_content支持的代码）
- onLayout():回调该方法来显示位置。在自定义ViewGroup中会用到，他调用的是子View的layout函数。onLayout一般是循环取出子View，然后经过计算得出各个子View位置的坐标值，然后用chid.layout(l,t,r,b)函数设置子View位置。
- onTouchEvent():监听到触摸事件回调，也是实现交互非常重要的回调方法
- onDraw():回调该方法对我们的控件进行绘制

注： onFinishInflite()或onAttachToWindow(): 可以得到子View对象

#### 二. 3种自定义View

- **扩展式：**

  扩展式自定义View继承自Android原生特定的View如：TextView，ImageView等等。我们通过重写onDrow()等回调方法对其进行**扩展**！使其实现我们想要的**功能或样式**！

  **注：**该方法实现的自定义View控件**不**需要自己支持wrap_content和padding。

- **组合式：**

  组合式自定义View继承自ViewGrop的子View如：LinearLayout、RelativieLayout等。当某种效果看起来像几种**View组合**在一起的时候，都可以使用这种方式实现。

  **注：**该方式实现自定义View**不**需要自己处理ViewGroup的测量和布局这两个过程。

- **完全自定义：**
  完全自定义View继承自View（android中所有控件的基类），通常实现一些不方便布局的组合方式来达到的，需要静态或动态地显示一些**不规则**的控件或图形！

  **注：** <u>该方法实现的自定义View控件**需要**自己支持wrap_content和padding。</u>

#### 三. 绘制流程-函数调用链

![绘制流程函数调用图](Custom-View-Components/cst_view_drawing_flowsheet.jpg)
#### 三. View生命周期

```flow
st=>start: 可见性改变 or Activity.onCreate()
op0=>operation: 实例化View-调用View构造方法
op1=>operation: View.onFinishInflate()
op2=>operation: View.onAttachedToWindow()
op3=>operation: View.onMeasure()
op4=>operation: View.onSizeChanged()
op5=>operation: onLayout()
op6=>operation: onDraw()
cond=>condition: Activity.onDestroy()
op=>operation: View.onDetachedFromWindow()
sub1=>subroutine: 子流程1
e=>end: 结束框
st->op0->op1->op2->op3->op4->op5->op6->cond
cond(yes)->op->e
cond(no)->e
```

## 四. 自定义步骤

1. 自定义View的属性
2. 在构造方法中获取自定义属性
3. 重写onMesure(可根据情况省略这一步)
4. 重写onDraw

注：

- 对于具有滑动效果的自定义View，还要做相关的滑动处理
- 如果遇到滑动冲突还需要解决相应的滑动冲突

第3步：(通常可以不重写该方法)

- 不重写：View这个类交给系统默认的处理
- 重写：如果View类的默认处理不满足我们的要求，我们就重写onMeasure函数

`wrap_content`或者是`match_parent`设置并没有指定真正的大小，可是我们绘制到屏幕上的View必须是要有具体的宽高，假如我们需要自定义一个正方形View这个时候必须对

protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){ … }

进行重写。

1和2就省略了，从第三步开始。

#### 五. 视需要重写onMeasure（设计宽高和测量模式）

widthMeasureSpec, heightMeasureSpec都是int型，一个int有4字节存储空间。

所以，对这4字节即32bit划分可以表示多个信息：最高位2bit用于区分测量模式(2的2次方=4，完全可以涵盖`UNSPECIFIED`，`EXACTLY`，`AT_MOST`这3种测量模式)，后面30bit存放宽或高的大小信息。

```
int widthMode = MeasureSpec.getMode(widthMeasureSpec);
int widthSize = MeasureSpec.getSize(widthMeasureSpec);
```

注：其实以上两个方法内部就进行bit位隔开取的计算了(移位`<<`和且`&`操作)

分析：此时获取的宽高的大小并不是最终绘制出来的大小，其大小来自父亲view的参考值。

| 测量模式    | 表示意思                                                    |
| ----------- | ----------------------------------------------------------- |
| UNSPECIFIED | 父容器没有对当前View有任何限制，当前View可以任意取尺寸      |
| EXACTLY     | 当前的尺寸就是当前View应该取的尺寸（固定值和 match_parent） |
| AT_MOST     | 当前尺寸是当前View能取的最大尺寸（ wrap_content）           |

**对wrap_content的支持**

在onMeasure()方法中实现对wrap_content的支持

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSpecSize= MeasureSpec.getSize(widthMeasureSpec);
    int heightSpectMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSpecSize=MeasureSpec.getSize(heightMeasureSpec);
    //这里就是对wrap_content的支持
    if(widthSpecMode==MeasureSpec.AT_MOST&&heightSpectMode==MeasureSpec.AT_MOST){
        //这里设定的根据你自己自定义View的情况而定
        setMeasuredDimension(200,200);
    }else if(widthSpecMode==MeasureSpec.AT_MOST){
        setMeasuredDimension(200,heightSpecSize);
    }else if (heightSpectMode==MeasureSpec.AT_MOST){
        setMeasuredDimension(widthSpecSize,200);
    }
}
```

注： 不要调用super.onMeasure( widthMeasureSpec, heightMeasureSpec);

**对padding的支持**

在onDraw()方法中实现对padding的支持，其实就是在绘制控件时考虑到padding就好了。如果不自己实现那么你对该自定义View设置padding将是无效的！

```java
@Override
   protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    //这里是对画一个圆形的View的padding支持
    final int paddingLeft = getPaddingLeft();
    final int paddingRight = getPaddingRight();
    final int paddingTop = getPaddingTop();
    final int paddingBottom=getPaddingBottom();
    int width = getWidth()-paddingLeft-paddingRight;
    int height = getHeight()-paddingBottom-paddingTop;
    int radius = Math.min(width,height)/2;
    canvas.drawCircle(paddingLeft+width/2,paddingTop+height/2,radius,mPaint_while);
}
```

#### 六. 重写onDraw（实现效果）

Canvas对象：画板

Canvas常用操作速查表：

| 操作分类     | 相关API                                                      | 备注                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 绘制颜色     | drawColor, drawRGB, drawARGB                                 | 使用单一颜色填充整个画布                                     |
| 绘制基本形状 | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧                |
| 绘制图片     | drawBitmap, drawPicture                                      | 绘制位图和图片)                                              |
| 绘制文本     | drawText, drawPosText, drawTextOnPath                        | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字 |
| 绘制路径     | drawPath                                                     | 绘制路径，绘制贝塞尔曲线时也需要用到该函数                   |
| 顶点操作     | drawVertices, drawBitmapMesh                                 | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用 |
| 画布剪裁     | clipPath, clipRect                                           | 设置画布的显示区域                                           |
| 画布快照     | save, restore, saveLayerXxx, restoreToCount, getSaveCount    | 依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 会滚到指定状态、 获取保存次数 |
| 画布变换     | translate, scale, rotate, skew                               | 依次为 位移、缩放、 旋转、错切                               |
| Matrix(矩阵) | getMatrix, setMatrix, concat                                 | 实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。 |
| 基础方法     | getDensity, getWidth, getHeight，getDrawFilter，isHardwareAccelerated(API 11)，getMaximumBitmapWidth，getMaximumBitmapHeight，getDensity，quickReject，isOpaque，setBitmap，setDrawFilter | 其他函数                                                     |

Path常用操作速查表：

| 作用            | 相关API                                                      | 备注                                                         |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 移动起点        | moveTo                                                       | 移动下一次操作的起点位置                                     |
| 设置终点        | setLastPoint                                                 | 重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同 |
| 连接直线        | lineTo                                                       | 添加上一个点到当前点之间的直线到Path                         |
| 闭合路径        | close                                                        | 连接第一个点连接到最后一个点，形成一个闭合区域               |
| 添加内容        | addRect, addRoundRect, addOval, addCircle, addPath, addArc, arcTo | 添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别) |
| 是否为空        | isEmpty                                                      | 判断Path是否为空                                             |
| 是否为矩形      | isRect                                                       | 判断path是否是一个矩形                                       |
| 替换路径        | set                                                          | 用新的路径替换到当前路径所有内容                             |
| 偏移路径        | offset                                                       | 对当前路径之前的操作进行偏移(不会影响之后的操作)             |
| 贝塞尔曲线      | quadTo, cubicTo                                              | 分别为二次和三次贝塞尔曲线的方法                             |
| rXxx方法        | rMoveTo, rLineTo, rQuadTo, rCubicTo                          | **不带r的方法是基于原点的坐标系(偏移量)， rXxx方法是基于当前点坐标系(偏移量)** |
| 填充模式        | setFillType, getFillType, isInverseFillType, toggleInverseFillType | 设置,获取,判断和切换填充模式                                 |
| 提示方法        | incReserve                                                   | 提示Path还有多少个点等待加入**(这个方法貌似会让Path优化存储结构)** |
| 布尔操作(API19) | op                                                           | 对两个Path进行布尔运算(即取交集、并集等操作)                 |
| 计算边界        | computeBounds                                                | 计算Path的边界                                               |
| 重置路径        | reset, rewind                                                | 清除Path中的内容 **reset不保留内部数据结构，但会保留FillType.** **rewind会保留内部的数据结构，但不保留FillType** |
| 矩阵操作        | transform                                                    | 矩阵变换                                                     |

Matrix常用操作速查表：

| 方法类别   | 相关API                                                | 备注                                         |
| ---------- | ------------------------------------------------------ | -------------------------------------------- |
| 基本方法   | equals hashCode toString toShortString                 | 比较、 获取哈希值、 转换为字符串             |
| 数值操作   | set reset setValues getValues                          | 设置、 重置、 设置数值、 获取数值            |
| 数值计算   | mapPoints mapRadius mapRect mapVectors                 | 计算变换后的数值                             |
| 设置(set)  | setConcat setRotate setScale setSkew setTranslate      | 设置变换                                     |
| 前乘(pre)  | preConcat preRotate preScale preSkew preTranslate      | 前乘变换                                     |
| 后乘(post) | postConcat postRotate postScale postSkew postTranslate | 后乘变换                                     |
| 特殊方法   | setPolyToPoly setRectToRect rectStaysRect setSinCos    | 一些特殊操作                                 |
| 矩阵相关   | invert isAffine(API21) isIdentity                      | 求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵 … |

贝塞尔曲线常用操作速查表：

| 贝塞尔曲线         | 对应的方法 | 演示动画                                                     |
| ------------------ | ---------- | ------------------------------------------------------------ |
| 一阶曲线(线性曲线) | lineTo     | [![img](https://user-gold-cdn.xitu.io/2017/8/23/05750d8554a308b440e7ebbfea88c525?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)](https://user-gold-cdn.xitu.io/2017/8/23/05750d8554a308b440e7ebbfea88c525?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) |
| 二阶曲线           | quadTo     | [![img](https://user-gold-cdn.xitu.io/2017/8/23/2a39d3cb8a36dbe8ddb3d068adffc00b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)](https://user-gold-cdn.xitu.io/2017/8/23/2a39d3cb8a36dbe8ddb3d068adffc00b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) |
| 三阶曲线           | cubicTo    | [![img](https://user-gold-cdn.xitu.io/2017/8/23/bd9b5ba0ef55d80f6ed3cc395ca95112?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)](https://user-gold-cdn.xitu.io/2017/8/23/bd9b5ba0ef55d80f6ed3cc395ca95112?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) |
| 四阶曲线           | 无         | [![img](https://user-gold-cdn.xitu.io/2017/8/23/bddf32dad9399c44dd232328ce56a2a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)](https://user-gold-cdn.xitu.io/2017/8/23/bddf32dad9399c44dd232328ce56a2a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) |

------

自定义View的延伸：**自定义ViewGroup**

设计思路:

> 1.首先，我们得知道各个子View的大小吧，只有先知道子View的大小，我们才知道当前的ViewGroup该设置为多大去容纳它们。
>
> 2.根据子View的大小，以及我们的ViewGroup要实现的功能，决定出ViewGroup的大小
>
> 3.ViewGroup和子View的大小算出来了之后，接下来就是去摆放了吧，具体怎么去摆放呢？这得根据你定制的需求去摆放了，比如，你想让子View按照垂直顺序一个挨着一个放，或者是按照先后顺序一个叠一个去放，这是你自己决定的。
>
> 4.已经知道怎么去摆放还不行啊，决定了怎么摆放就是相当于把已有的空间”分割”成大大小小的空间，每个空间对应一个子View，我们接下来就是把子View对号入座了，把它们放进它们该放的地方去。

步骤：(实现ViewGroup垂直摆放child view)

1. 重写onMeasure，实现测量子View大小以及设定ViewGroup的大小

```java
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //将所有的子View进行测量，这会触发每个子View的onMeasure函数
        //注意要与measureChild区分，measureChild是对单个view进行测量
        measureChildren(widthMeasureSpec, heightMeasureSpec);

        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);

        int childCount = getChildCount();

        if (childCount == 0) {//如果没有子View,当前ViewGroup没有存在的意义，不用占用空间
            setMeasuredDimension(0, 0);
        } else {
            //如果宽高都是包裹内容
            if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
                //我们将高度设置为所有子View的高度相加，宽度设为子View中最大的宽度
                int height = getTotleHeight();
                int width = getMaxChildWidth();
                setMeasuredDimension(width, height);

            } else if (heightMode == MeasureSpec.AT_MOST) {//如果只有高度是包裹内容
                //宽度设置为ViewGroup自己的测量宽度，高度设置为所有子View的高度总和
                setMeasuredDimension(widthSize, getTotleHeight());
            } else if (widthMode == MeasureSpec.AT_MOST) {//如果只有宽度是包裹内容
                //宽度设置为子View中宽度最大的值，高度设置为ViewGroup自己的测量值
                setMeasuredDimension(getMaxChildWidth(), heightSize);

            }
        }
    }
    /***
     * 获取子View中宽度最大的值
     */
    private int getMaxChildWidth() {
        int childCount = getChildCount();
        int maxWidth = 0;
        for (int i = 0; i < childCount; i++) {
            View childView = getChildAt(i);
            if (childView.getMeasuredWidth() > maxWidth)
                maxWidth = childView.getMeasuredWidth();

        }

        return maxWidth;
    }

    /***
     * 将所有子View的高度相加
     **/
    private int getTotleHeight() {
        int childCount = getChildCount();
        int height = 0;
        for (int i = 0; i < childCount; i++) {
            View childView = getChildAt(i);
            height += childView.getMeasuredHeight();

        }

        return height;
    }
```

1. 摆放child View

```java
@Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int count = getChildCount();
        //记录当前的高度位置
        int curHeight = t;
        //将子View逐个摆放
        for (int i = 0; i < count; i++) {
            View child = getChildAt(i);
            int height = child.getMeasuredHeight();
            int width = child.getMeasuredWidth();
            //摆放子View，参数分别是子View矩形区域的左、上、右、下边
            child.layout(l, curHeight, l + width, curHeight + height);
            curHeight += height;
        }
    }
```

参看文章：

[Developers](https://developer.android.com/guide/topics/ui/custom-components)

[自定义view总结](https://juejin.im/post/599d2b2e518825242238d4f6)

[自定义View之总结](https://www.jianshu.com/p/bcae9ec222e6)

[自定义View，有这一篇就够了](https://www.jianshu.com/p/c84693096e41)

[自定义View（三）—自定义View整个流程的梳理与总结 - 李诗雨](https://blog.csdn.net/cjm2484836553/article/details/71024436)

引申：

[GcsSloop](http://www.gcssloop.com/#blog)

[自定义View分类与流程](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B01%5DCustomViewProcess.md)

[Android 自定义View (二) 进阶 - Hongyang)](http://blog.csdn.net/lmj623565791/article/details/24300125)

[Android: draw a custom view - Roman Danylyk (Medium)](https://medium.com/@romandanylyk96/android-draw-a-custom-view-ef79fe2ff54b)

[HenCoder Android 开发进阶: 自定义 View 1-1 绘制基础](http://hencoder.com/ui-1-1/)

[深度解析View构造函数中的参数defStyleAttr](https://www.jianshu.com/p/08be3c08c576)

[View的生命周期方法和Activity生命周期方法关系](https://blog.csdn.net/lue2009/article/details/45692009)