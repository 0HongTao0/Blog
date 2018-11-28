---
title: Android 绘制过程
date: 2018-08-02 10:01:25
tags:
---

##### Android 的 setContentView 流程

- （1）Activity 是 Android 中最直接的，也是最主要的显示界面，在 Activity 中，要通过 setContentView() 来设置当前 Activity 中的布局 Layout。

- （2）setContentView() 是干嘛的？翻译过来，就是设置内容的 View 。如何设置？通过 Window 类来进行设置。通过源码发现，Activity 的 setContentView 方法实际上是调用 Window 的 setContentView 方法。

  ```java
  public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
  }
  ```

- （3）但是呢，Window 是一个抽象类，具体需要去找出它的实现类。Window 是什么？官方一点来讲就是处理顶级窗口外观和行为策略的抽象基类。通俗来理解就是 Activity 通过 Window 来对 View 进行一系列的处理。并且通过源码的解释可以知道该抽象类的唯一实现类是 PhoneWindow.java 。

  ```java
  /**
   * Abstract base class for a top-level window look and behavior policy.  An
   * instance of this class should be used as the top-level view added to the
   * window manager. It provides standard UI policies such as a background, title
   * area, default key processing, etc.
   *
   * <p>The only existing implementation of this abstract class is
   * android.view.PhoneWindow, which you should instantiate when needing a
   * Window.
   */
  ```

- （4）通过 Window 的注释，我们知道该类的唯一实现类就是 PhoneWindow ，并且 Activity 的 setContentView 方法实际是调用 Window 的 setContentView 方法，所以要去找 Window 的实现类 PhoneWindow 的 setContentView 方法。发现 PhoneWindow 中的 setContentView 中有 3 个重载方法，不过根据入参，需要找的是具有一个 int 参数的 setContentView 方法。

  ```java
  @Override
  public void setContentView(int layoutResID) {
    if (mContentParent == null) {
      installDecor(); //注意此方法，下面会提到。
    } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
      mContentParent.removeAllViews(); // 删除所有子 View
    }
  
    if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
      final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                                                     getContext());
      transitionTo(newScene);
    } else {
      mLayoutInflater.inflate(layoutResID, mContentParent);//布局 layout 在方法中的使用
    }
    mContentParent.requestApplyInsets();
    final Callback cb = getCallback();
    if (cb != null && !isDestroyed()) {
      cb.onContentChanged();
    }
    mContentParentExplicitlySet = true;
  }
  ```

  由 View inflate(@LayoutRes int resource, @Nullable ViewGroup root) 方法可以大概知道，是将对应 layoutResID 的布局加载到 mContentParent 中去。

- （5）installDecor 方法其实在 PhoneWindow 的 setContentView 方法中首先调用，由方法名翻译出来是：设置装饰，设置布局好一点（根据代码意思）。首先调用 installDecor 方法，说明先初始化 mDecor 的布局，所以要先理解一下 mDecor 是什么，才能知道 mContentParent 是对应哪个 View，以及它的层次结构。

  ```java
  private void installDecor() {
   mForceDecorInstall = false;
   if (mDecor == null) {
     mDecor = generateDecor(-1); // mDecor 的初始化,在此会 new 出一个 Decor 对象返回
     mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
     mDecor.setIsRootNamespace(true);
     if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
       mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
     }
   } else {
     mDecor.setWindow(this);
   }
   if (mContentParent == null) {
     mContentParent = generateLayout(mDecor); // mContentParent 的初始化
  
     mDecor.makeOptionalFitsSystemWindows();
  
     final DecorContentParent decorContentParent = (DecorContentParent) mDecor.findViewById(
       R.id.decor_content_parent);
     //...省略下面源码
  }
  ```

  初始化 mDecor，Decor 根据成员属性的解释知道是 window 的最顶层的 view（top-level）。由 mContentParent = generateLayout(mDecor); 代码自然地知道 mContentParent 的构造依赖于 Decor，并且 Decor 是最顶层的 View，所以自然推出 mContentParent 是 Decor 的子 View。

  ```java
  // This is the top-level view of the window, containing the window decor.、
  private DecorView mDecor;
  
  protected DecorView generateDecor(int featureId) {
   Context context;
   if (mUseDecorContext) {
     Context applicationContext = getContext().getApplicationContext();
     if (applicationContext == null) {
       context = getContext();
     } else {
       context = new DecorContext(applicationContext, getContext().getResources());
       if (mTheme != -1) {
         context.setTheme(mTheme);
       }
     }
   } else {
     context = getContext();
   }
   // new 出一个 DecorView 对象，构造方法中调用 super.(context)。则会对 Decor 进行测量布局绘制等过程。
   return new DecorView(context, featureId, this, getAttributes());
  }
  ```

  初始化 mContentParent 的代码其实就是加载好 Decor 后在 Decor 中利用 findViewById 寻找到 mContentParent 的值并返回。不过要经历一大堆的处理，比如系统版本，App Theme 的配置等（具体不再深入）。

  ```java
  protected ViewGroup generateLayout(DecorView decor) {
    // 省略的代码大概就是读取 window 的 theme 属性参数进行配置设置
    //... 源码有点长 ，省略一点
    // 并且在 window 中把 decor 加载进来
  
    // Inflate the window decor.
    int layoutResource;
    int features = getLocalFeatures();
    // 根据 features 的值进行逻辑判断，加载特定的 layoutResource （Decor 的标题 Layout 布局文件 Id）
    mDecor.startChanging();
    mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);// 加载 Decor 的布局文件
  
    // 这就是我们 setContentView 的布局文件加载到的父 ViewGroup 中（内容栏）
    ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
    if (contentParent == null) {
      throw new RuntimeException("Window couldn't find content container view");
    }
  
    if ((features & (1 << FEATURE_INDETERMINATE_PROGRESS)) != 0) {
      ProgressBar progress = getCircularProgressBar(false);
      if (progress != null) {
        progress.setIndeterminate(true);
      }
    }
    //... 源码有点长 ，省略一点
  }
  ```

- 通过上面代码的追溯，可以找到 mContentParent 的父 View 是 Decor，以及 mContentParent 是在哪赋值的。现在再回到第（4）步中就非常清晰了，再 nstallDecor 方法调用后，mContentParent 就已经被赋值，并且 Decor（Window 根 Vew）已经被加载到 Window 中。再通过 LayoutInflater 将我们在 Activity 中设置的布局加载到 mContentParent 中。关于 Decor 的布局根据系统版本的不同而不同。

##### 简单介绍一下 Measure 的重要参数值代表的意思。

- MeasureSpec ：翻译过来就是测量规格，一个 View 的 MeasureSpec 计算数据由父 View 传下来的 MeasureSpec 和自己本身的 LayoutParams（xml 文件配置的 layout 属性的标签，如：layout_width 和 layout_height） 决定。MeasureSpec 是一个 32 位的 int 值，存储方式是以**二进制存储**，高 2 位代表 SpecMode（测量模式），低 30 位代表是 SpecSize（规格大小）。

- SpecMode 对 View 测量规格的影响如下。父 View 的测量过程会先测量子 View，是一个树形的的递归测量实现，所以先测量完子 View，等到没有子 View 的时候在测量父 View 本身。

  

  |                     父 View 的 SpecMode                      |               子 View 的 LayoutParams               |                 子 View 的 Size 和 Mode 情况                 |
  | :----------------------------------------------------------: | :-------------------------------------------------: | :----------------------------------------------------------: |
  | <br />EXACTLY<br />父 View 已经检测到子 View 的大小，最终大小就是 SpecSize | 1. 确切值<br />2. match_parent<br />3. wrap_content | 1. size = 计算出来的值，mode = EXACTLY<br />2. size =  父 View size ，mode = EXACTLY<br />3. size =  父 View size ，mode = AT_MOST |
  |      <br />AT_MOST<br />父容器指定子 View Size 的最大值      | 1. 确切值<br />2. match_parent<br />3. wrap_content | 1. size = 计算出来的值，mode = EXACTLY<br />2. size =  父 View size ，mode = AT_MOST<br />3. size =  父 View size ，mode = AT_MOST |
  |     <br />UNSPECIFIED<br />父容器对子 View 的大小没限制      | 1. 确切值<br />2. match_parent<br />3. wrap_content | 1. size = 计算出来的值，mode = EXACTLY<br />2. size =  0 ，mode = UNSPECIFIED<br />3. size =  0，mode = UNSPECIFIED |

  总结一下：

  - 子 View 的 LayoutParams 是确切的，那么无论父 View 是 specMode 是怎样的，子 View 的 size 都是开发设置的确切值，子 View 的 size 被确定，所以子 View 的 specMode = EXACTLY；
  - 子 View 的 LayoutParams = match_parent（充满父 View）。当父 View 的 specMode 是 EXACTLY ，父 View 的 size 已经是确定的，那么子 View 的 size = 父 View 的 size，子 View 的 size 被确定，所以子 View 的 specMode = EXACTLY。当父 View 的 specMode 是 AT_MOST，也就是说父 View 的 size 是不确切的，但是父 View 有个最大 size 值，具体 size 需要根据子 View 的 size 确定，子 View 要充满父 View，应该子 View 的 size = 父 View 的 size，父 View 的大小不确定，子 View 要充满父 View，所以子 View 的 size 不确定，specMode 也应该是 AT_MOST。当父 View 的 specMode = UNSPECIFIED ，没有限制，不可能把子 View 的 size 设置成无限大，所以暂时设置成 0 ，specMode = UNSPECIFIED，父 View 无限制，父 View 的 size 无确切值和最大值，子 View 充满父 View，子 View 的 specMode 也应该是 UNSPECIFIED。 
  - 子 View 的LayoutParams = wrap_content（包含内容的大小）。当父 View 的 specMode = EXACTLY 或 AT_MOST 时，不管父 View 的大小是否确定，在这 2 种 specMode 中，都有一个确切值或者最大值，子 View 的大小要根据内容决定，子 View 的 size 大小又不能超过父 View，所以子 View 的 size 肯定是一个不确定的值，specMode 应该是 AT_MOST，size 应该是父 View 的确切值或者是最大值。当父 View 的 specMode = UNSPECIFIED，没有限制，不可能把子 View 的 size 设置成无限大，所以暂时设置成 0 ，父 View 无限制，父 View 的 size 无确切值和最大值，子 View 的 size 是由子 View 的内容决定，子 View 的 specMode 也应该是 UNSPECIFIED。

  计算 MeasureSpec 的源代码实现如下：

  ```java
  public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
      int specMode = MeasureSpec.getMode(spec);
      int specSize = MeasureSpec.getSize(spec);
      int size = Math.max(0, specSize - padding);
      int resultSize = 0;
      int resultMode = 0;
      switch (specMode) {
          case MeasureSpec.EXACTLY:
              if (childDimension >= 0) {
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  resultSize = size;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              }
              break;
          case MeasureSpec.AT_MOST:
              if (childDimension >= 0) {
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              }
              break;
          case MeasureSpec.UNSPECIFIED:
              if (childDimension >= 0) {
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                  resultMode = MeasureSpec.UNSPECIFIED;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                  resultMode = MeasureSpec.UNSPECIFIED;
              }
              break;
      }
      return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
  }
  ```

##### 从 DecorView 到其子 View 的 Measure 过程

- 由上面分析知道，一个 Activity 的最顶级的 View 就是 DecorView，那么测量过程就应该从 DecorView 以树形结构往 DecorView 的子 View 开始递归实现。对于 DecorView ，它的 MeasureSpec 不同普通 View 那样决定，它的决定是由窗口的尺寸和自身的 LayoutParams 来决定（普通 View 的 MeasureSpec 的决定由父 View 的 MeasureSpec 和自身的 LayoutParams）。那么 DecorView 什么时候开始测量？上面提到，在 PhoneWindow 中的 setContentView 方法中调用 installDecor 方法，在 installDecor 方法里面会初始化一个 DecorView，初始化一个 View ，自然要对 View 进行测量。

- View 的测量是由 measure 方法工作的，measure 方法是 final ，说明此方法不能重写，而在 measure 方法中最终会调用 onMeasure 方法，所以要看 onMeasure 方法。

  ```java
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
      setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                           getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
  }
  ```

  setMeasuredDimension 会设置 View 的宽/高的测量值。在 View 的子类中，要计算出 widthMeasureSpec 和 heightMeasureSpec 的实际值，再调用 super.measure 方法或者直接调用 setMeasuredDimension 方法设定 View 的宽/高测量值。

  getDefaultSize 方法是干嘛的呢？返回 View 测量后的大小。

  ```java
  public static int getDefaultSize(int size, int measureSpec) {
      // View 的默认大小。
      //View 无背景：由 android_minXXX 决定，View 有背景：android_minXXX 和 背景的宽/高取最大值
      int result = size; 
      int specMode = MeasureSpec.getMode(measureSpec);// 测量得出的 View 的 mode
      int specSize = MeasureSpec.getSize(measureSpec);// 测量得出的 View 的 size
      switch (specMode) {
          case MeasureSpec.UNSPECIFIED:
              result = size;
              break;
          case MeasureSpec.AT_MOST:
          case MeasureSpec.EXACTLY:
              result = specSize;
              break;
      }
      return result;
  }
  ```

  **在自定义 View 中，要注意重写 onMeasure 方法，并设置 wrap_content 时的自身大小。**

  当 View 的 Layout_Params = wrap_content 时，由上面的表格知道，View 的 specMode 是 AT_MOST，由 getDefaultSize 方法逻辑知道，计算的最终大小是 specSize，而这个 specSize 就是父 View 的可使用的的大小，子 View 的 specSize = 父 View 的 specSize，就如同 match_parent 的效果。

  避免上面问题，重写 onMeasure 方法如下：

  ````java
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
      super.onMeasure(widthMeasureSpec, heightMeasureSpec);
      int width; 	// view 的默认宽度
      int height; // view 的默认高度
      
      int widthSpecMode = mewsureSpec.getMode(widthMeasureSpec);// 父 View 的宽度 specMode
      int widthSpecSize = mewsureSpec.getSzie(widthMeasureSpec);// 父 View 的宽度 specSize
      int heightSpecMode = mewsureSpec.getMode(heightMeasureSpec);// 父 View 的高度 specMode
      int heightSpecSize = mewsureSpec.getSzie(heightMeasureSpec);// 父 View 的高度 specSize
      if (widthSpecMode == MeasureSpec.AT_MOST && heightSpecMode == MeasureSpec.AT_MOST){
          setMeasuredDimension(width, height);
      } else if (widthSpecMode == MeasureSpec.AT_MOST){
          setMeasuredDimension(width, heightSpecSize);
      } else if (heightSpecMode == MeasureSpec.AT_MOST){
          setMeasuredDimension(widthSpecSize, height);
      }
  }
  ````

- ViewGroup 的测量除了要测量自己，还要遍历它所有子 View 的 measure 过程，最后完成测量过程。具体遍历子 View 的过程是由 measureChildren 方法实现，调用子 View 的测量方法是 measureChild 方法实现。

  ```java
  protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
      final int size = mChildrenCount;
      final View[] children = mChildren;
      for (int i = 0; i < size; ++i) {// 遍历子 View
          final View child = children[i];
          if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
              measureChild(child, widthMeasureSpec, heightMeasureSpec);
          }
      }
  }
  protected void measureChild(View child, int parentWidthMeasureSpec,
              int parentHeightMeasureSpec) {
      final LayoutParams lp = child.getLayoutParams();
      final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                  mPaddingLeft + mPaddingRight, lp.width);
      final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                  mPaddingTop + mPaddingBottom, lp.height);
      child.measure(childWidthMeasureSpec, childHeightMeasureSpec); // 调用子 View 的测量方法
  }
  ```

  

- 当调用完 setMeasuredDimension ，将测量的值传入，测量的过程就完成了。