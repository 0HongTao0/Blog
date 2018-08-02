---
title: Android setContentView
date: 2018-08-02 10:01:25
tags:
---

##### Android 的绘制流程

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
     mDecor = generateDecor(-1); // mDecor 的初始化
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
