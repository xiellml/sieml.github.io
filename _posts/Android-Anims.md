---
title: Android Animation
date: 2017-12-23 11:05:27
tags: animation  
---


Android动画共有**3**大类
---
*  逐帧动画, 简称"帧动画", 英文名: Frame Animation or Drawable Animation
*  补间动画, 又称"视图动画", 英文名: Tween Animation or View Animation
*  属性动画, 英文名: Property Animation


区别
---
> API 3.0之后才新加入了Property动画作为补充
> 凡是前两种动画可以实现的效果,  属性动画一定可以实现其效果
> 如果你想在API < 3.0的机器上使用同样效果, 可使用NineOldAndroids动画库

1. **逐帧动画最简单**  一帧一帧的, 可以使用XML文件把资源组合起来
2. **补间动画最常见**  补间补间, 就是把起止帧之间的帧画出来, 改变View的透明度, 位置, 大小, 角度.  它结束后看到的位置不是真实坐标的改变, 只是绘制位置的改变.
3. **属性动画最强大**  会改变View的真实属性(坐标), 父类为系统抽象类Animator(有500多行)基本不直接使用, 最常用的是子类ValueAnimator和ObjectAnimator.

区别：

（1）属性动画比视图动画更强大，不但可以实现缩放、平移等操作，还可以自己定义动画效果，监听动画的过程，在动画过程中或完成后做响应的动作。

（2）属性动画不但可以作用于View，还能作用于Object。

（3）属性动画利用属性的改变实现动画，而视图动画仅仅改变了view的大小位置，但view真正的属性没有改变。

注：关于真实属性，比如视图动画作用平移之后点击它会无效，因为它的坐标并未真正改变需要点击到原来位置才能生效，而这个问题对于属性动画是不存在的。


使用
---
### 1. 逐帧动画
#### 特点: 
*细腻灵活*  	*资源制作量大*  	*图片文件多*

#### 使用步骤: 
1. 添加帧: XML定义资源文件, 或者Java代码创建 
2. 引用支持类: AnimationDrawable
#### 代码: 
1. XML方式
```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list
   xmlns:android="http://schemas.android.com/apk/res/android" 
   android:oneshot="false"  >
      <!-- 定义一个动画帧，Drawable为img0，持续时间50毫秒 -->
      <item android:drawable="@drawable/img0" android:duration="50" />
      <!-- 定义一个动画帧，Drawable为img1，持续时间100毫秒 -->
      <item android:drawable="@drawable/img1" android:duration="100" />
  </animation-list>

//然后加载XML
frameAnim =(AnimationDrawable)getResources().getDrawable(id);
view.setBackgroundDrawable(frameAnim);
frameAnim.start();frameAnim.stop();
```
2. Java代码方式
```Java
frameAnim =new AnimationDrawable();
frameAnim.addFrame(getResources().getDrawable(R.drawable.img0), 50);
frameAnim.addFrame(getResources().getDrawable(R.drawable.img1), 100);
frameAnim.setOneShot(false);view.setBackgroundDrawable(frameAnim);
frameAnim.start();
frameAnim.stop();
```

### 2. 补间动画
#### 原理:
给出两个关键帧, 通过一些算法将给定属性值在给定的时间内在两个关键帧间渐变。

方式实现类:

- AlphaAnimation：透明度（alpha）渐变效果，对应标签。
- TranslateAnimation：位移渐变，需要指定移动点的开始和结束坐标，对应标签。
- ScaleAnimation：缩放渐变，可以指定缩放的参考点，对应标签。
- RotateAnimation：旋转渐变，可以指定旋转的参考点，对应标签。
- AnimationSet：组合渐变，支持组合多种渐变效果，对应标签

#### 特点: 
*过渡自然*  	*实现简单快速* 

#### 使用步骤: 
1. 设置动画形式: 如渐变 
2. 设定起止状态: 如起始1.0f和结束0.1f

#### 代码: 
1. XML方式
  XML文件的根元素可以为<alpha>, <scale>, <translate>, <rotate>,<interpolator>, <set>(表示以上几个动画的集合, set可以嵌套). 默认情况下, 所有动画是同时进行的, 可以通过startOffset属性设置各个动画的开始偏移（开始时间）来达到动画顺序播放的效果.
```xml
<alpha xmlns:android="http://schemas.android.com/apk/res/android"      android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    android:fromAlpha="1.0"  
    android:toAlpha="0.1"  
    android:duration="2000"
/>  
```
2. Java代码方式
```Java
Animation anim = new AlphaAnimation(1.0f,0.1f);
//设置持续时间
anim.setDuration(2000);
//清除原有的动画，避免多次点击出现重复的效果
view.clearAnimation();
//开始执行动画
view.startAnimation(anim);
```

### 3. 属性动画
#### 特点: 
*实现费脑* 	*属性接口更多* (如改变view的背景颜色属性)

#### 使用步骤: 
1. 直接new出ValueAnimator或者子类ObjectAnimator 
2. 设置动画到View

#### 代码: 

1. XML方式:

```xml
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:ordering="together">
<objectAnimator
    android:valueType="floatType"
    android:propertyName="scaleX"
    android:valueFrom="1"
    android:valueTo="2"/>
<objectAnimator
    android:propertyName="translationX"
    android:valueType="floatType"
    android:valueFrom="0"
    android:valueTo="200"/>
</set>

Animator animator= AnimatorInflater.loadAnimator(this,R.animator.animator);
animator.setTarget(imageView);
animator.start();
```

2. 代码方式: 

支持rotation, alpha, translationX, translationY, scaleX, scaleY, 以及使用AnimatorSet组合前者多种基础动画方式
```Java
ObjectAnimator animator = ObjectAnimator.ofFloat(imageView, "translationX", 0f, -300f, 0f);
animator.setDuration(2000);
animator.start();
```
AnimationSet提供了一个把多个动画组合成一个组合的机制，并可设置组中动画的时序关系，如同时播放，顺序播放等。 
```Java
AnimatorSet bouncer = new AnimatorSet(); 
bouncer.play(anim1).before(anim2); 
bouncer.play(anim2).with(anim3); 
bouncer.play(anim2).with(anim4) 
bouncer.play(anim5).after(amin2); 
animatorSet.start();
```
#### 子类源码剖析: ObjectAnimator继承自ValueAnimator

**ValueAnimator** 包含动画的开始值，结束值，持续时间等属性
- 内部流程:
1. 计算属性值
  时间因子(input) = 动画已进行的时间跟动画总时间(duration)的比计算(0~1)—TimeInterpolator 
  插值因子(fraction,表示动画的完成度) = TimeInterpolator通过开始值,结束值,时间因子计算出—TimeInterpolator 
  公式为: fraction = (Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f; 
  属性值 = TypeAnimator通过插值因子计算出 = {开始值,结束值,差值}—TypeAnimator 
  根据TypeEvaluator的函数evaluate(): 
  public Float evaluate(float fraction, Number startValue, Number endValue) { 
  float startFloat = startValue.floatValue(); 
  return startFloat + fraction * (endValue.floatValue() - startFloat); 
  }

2. 根据属性值执行相应的动作, 如改变对象的某一属性。
  需要实现ValueAnimator.onUpdateListener接 口，这个接口只有一个函数onAnimationUpdate()， 
  在这个函数中会传入ValueAnimator对象做为参数，通过这个 ValueAnimator对象的getAnimatedValue()函数可以得到当前的属性值 
  ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f); 
  animation.setDuration(1000); 
  animation.addUpdateListener( AnimatorUpdateListener() { 
  @Override 
  onAnimationUpdate(ValueAnimator animation) { 
  Log.i(“update”, ((Float) animation.getAnimatedValue()).toString()); 
  } 
  }); 
  animation.setInterpolator( CycleInterpolator(3)); 
  animation.start();

**ObjectAnimator** 属性动画一般使用这个类, 但需满足以下条件, 否则就选用ValueAnimator
- 这也是两者的区别:
1. 对象(View类等)应该有一个setter函数：set（驼峰命名法） 
2. 如上面的例子中，像ofFloat之类的工场方法，第一个参数为对象名，第二个为属性名，后面的参数为 
  可变参数，如果values…参数只设置了一个值的话，那么会假定为目的值，属性值的变化范围为当前值到目的值，为了获得当前值， 
  该对象要有相应属性的getter方法：get（驼峰命名法） 
3. 如果有getter方法，其应返回值类型应与相应的setter方法的参数类型一致。

- 内部流程: 
  同ValueAnimator. 根据应用动画的对象或属性的不同, 可能需要在onAnimationUpdate函数中调用invalidate()函数刷新视图。 

说明：请求重绘View树，即draw()过程，假如视图发生大小没有变化就不会调用layout()过程，并且只绘制那些“需要重绘的” 视图，即谁(View的话，只绘制该View ；ViewGroup，则绘制整个ViewGroup)请求invalidate()方法，就绘制该视图。 

#### 属性动画扩展之LayoutTransition:
> 容器布局动画就是当一个布局容器中的view方式改变时所产生的动画， 
> 比如：但一个相对布局中新增加一个view时或者删除一个view时(或者visible改变时)，那么就可以通过一个动画来进行表现。

- 5种状态变化:
1. APPEARING: 动画所运行的项目出现在这个容器中时，即：view显示时的动画 
2. CHANGE_APPEARING: 由于在这个容器总新增加了一个view，而导致原来的view位置发生改变所以会触发这个动画。 
3. DISAPPEARING: view在这个容器中消失时触发的动画
4. CHANGE_APPEARING：其他视图的出现导致某个视图改变。
5. CHANGE_DISAPPEARING：其他视图的消失导致某个视图改变。
- 使用步骤：
1. 创建LayoutTransition对象mTransitioner 
2. 创建动画 
3. 在xml文件中将相应布局的属性Android:animateLayoutChanges=”true”设置为true 
4. 将动画通过mTransitioner的setAnimator方法设置给mTransitioner 
5. 通过布局控件的setLayoutTransition方法将mTransitioner设置进去

- 代码
```Java
mButtonAddBtn = (Button) findViewById(R.id.buttonLayoutAnmation);
        mLayout = (LinearLayout) findViewById(R.id.layoutanmation);
        LayoutTransition transition = new LayoutTransition();//创建LayoutTransition对象
        transition.getDuration(2000);
        transition.setAnimator(LayoutTransition.APPEARING,  AnimatorInflater.loadAnimator(getApplicationContext(), R.animator.animator));//创建动画
        transition.setAnimator(LayoutTransition.CHANGE_APPEARING, null);
        transition.setAnimator(LayoutTransition.DISAPPEARING, null);
        transition.setAnimator(LayoutTransition.CHANGE_DISAPPEARING,null);
        mLayout.setLayoutTransition(transition);//方法将mTransitioner设置给布局
        mButtonAddBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                count++;
                Button btn = new Button(ActivityLayoutAnimations.this);//1.建立button对象
                //2.设置LayoutParams，宽和高
                ViewGroup.LayoutParams params = new ViewGroup.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
                btn.setLayoutParams(params);//3.将宽高加到button上
                btn.setText("按钮" + count);
                mLayout.addView(btn);//4.将button加到布局中
               }
        });
```

后记
---
1. 高级用法:http://blog.csdn.net/jdsjlzx/article/details/45558901 
  属性动画的高级用法中最有技术含量的也就是如何编写出一个合适的**TypeEvaluator**

2. View绘制流程: 
  onMeasure()->onLayout()->onDraw() 
  即是绘制: 大小->位置->绘制 
  整个View树的绘图流程在ViewRoot.java类的performTraversals()函数展开，该函数所做 的工作可简单概况为是否需要重新计算视图大小(measure)、是否需要重新安置视图的位置(layout)、以及是否需要重绘(draw)