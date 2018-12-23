title: Android 基础

date: 2018-12-13 14:00

tags:

------
### IPC 机制（Inter-Process Communication）
  参考：《Android 开发艺术探索》
  - Android 为什么需要跨进程通信？
    Android 是基于 Linux 系统的，在 Linux 系统中进程之间是相互独立的（进程隔离），为了保证安全性和独立性，一个进程不能直接访问另一个进程的空间。
  - Android 多进程：在 Android 中使用多进程的唯一方法是给四大组件在 AndroidMenifest 中指定 android:process 属性。
  - Android 多进程带来的问题：
    1. 静态成员和单例模式完全失效
    2. 线程同步机制完全失效
    3. SharedPreferences 的可靠性下降（不支持 2 个进程同时写入）
    4. Application 多次创建
    **不同进程属于不同的虚拟机和不同的 Application，拥有不同的内存空间**
  - 序列化
    1. Serializable 接口：JAVA 提供的，耗能较大，通常用于储存设备序列化和网络传输序列化。
    2. Parcelable 接口：Android 提供的，效率高，主要用于内存的序列化。（Android Studio 有插件生成）
  - AIDL 实现跨进程通信
    ![AIDL 过程图](https://github.com/0HongTao0/Blog/blob/master/pic/AIDL%20Binder.jpg?raw=true)

<!--more-->

---
### [Android 内存优化](https://mp.weixin.qq.com/s/Z7oMv0IgKWNkhLon_hFakg?)
  - **RAM优化** ：降低程序在运行时使用的内存，防止由于内存不足导致程序被系统杀死。（LMK 机制：LowMemoryKill）
  - ROM 优化 : 主要是降低程序的占用空间，降低 Apk 的大小，防止偶遇 ROM 不足导致程序无法被安装。

  - 内存泄漏的解决方法
    1. [AndroidExcludedRefs](https://github.com/square/leakcanary/blob/master/leakcanary-android/src/main/java/com/squareup/leakcanary/AndroidExcludedRefs.java) 列出很多由于系统原因导致引用无法释放的例子，通过 hack 的建议去修复。
    2. 兜底回收内存，Activity 泄漏导致其引用的资源无法释放，兜底回收是指将 Activity 所持有的资源都回收掉（引用置 null），然后剩下的 Activity 就是个空壳。
  - 降低运行时内存大小
    1. **减少 bitmap 占用的内存，图片按需求的 View 大小加载，监控重复图片。统一使用 bitmap 的加载器，在发生 OOM 时清除缓存。**
    2. 监控程序堆内存的使用率，达到程序的设定值则启动相关模块进行内存释放。
    3. 多进程：将对于会引发内存泄漏或者占用内存过大的组件放置单独的进程中运行。
  - **[内存优化策略](https://time.geekbang.org/column/article/71610)**
    并不是内存占用越少越好，而是要通过设备环境来进行综合考虑，在设备内存比较充足的环境下，尽可能地提供更好的用户体验；而在设备内存比较少的情况下则需要保证程序的正常运行，不让程序由于 OOM 而 Crash。
    1. 设备分级，低端机用户关闭耗内存的功能，比如动画，缓存等。
    2. 缓存管理，建立一套完善的缓存管理机制，当系统内存不足时应该释放内存。
    3. 减少进程数，空进程也会占用 10M 的内存。尤其是应用启动的进程还有保活进程。
    4. 降低 APK 的大小。

---
### [Android Retrofit](https://square.github.io/retrofit/)
  - Retrofit 是一个网络请求的封装库，为什么说它只是个封装库，因为它本身并没有实现网络请求的逻辑，而是底层交给 OkHttp 进行实现网络请求，支持多种数据转换器（Converters，如 GSON, Jackson）。
  - 使用特性：
    1. 通过接口注解来描述一个 HTTP 请求，Url 参数和 Request 参数。
    2. 通过定义返回值为 Observable<> 类型，它还能异步框架 RxJava 结合，实现异步请求网络进行加载数据。
    3. 支持获取原生的 HTTP 请求数据（Raw）。
  - 源码实现
    1. Retrofit 实例化是通过 Builder 设计模式来实现的，可以配置 Retrofit 的 **根 URL**， **数据转换器**， **进行 HTTP 网络请求的客户端**。
    2. 通过一个接口和接口方法上的注解来定义一个 HTTP 网络请求的 API 接口，这个接口包含 HTTP 请求的类型,参数等信息。
    3. 在底层 creat() 方法，Retrofit 通过 JAVA 动态代理来获取 HTTP 请求接口的代理对象，代理对象将 HTTP 请求接口方法上面的注解解析成一个 ServiceMethod 类型的对象，再通过该对象构建 OKHttp 的请求。
    ```Java
    public <T> T create(final Class<T> service) {
        Utils.validateServiceInterface(service);
        if (validateEagerly) {
          eagerlyValidateMethods(service);
        }
        return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
            new InvocationHandler() {
              private final Platform platform = Platform.get();

              @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
                  throws Throwable {
                // If the method is a method from Object then defer to normal invocation.
                if (method.getDeclaringClass() == Object.class) {
                  return method.invoke(this, args);
                }
                if (platform.isDefaultMethod(method)) {
                  return platform.invokeDefaultMethod(method, service, proxy, args);
                }
                ServiceMethod<Object, Object> serviceMethod =
                    (ServiceMethod<Object, Object>) loadServiceMethod(method);
                OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
                return serviceMethod.adapt(okHttpCall);
              }
            });
      }
    ```
---
### Content Provider

- [Content Provider 的一些理解](https://www.jianshu.com/p/c70ae80cf64d)
  1. 主要用于不同程序之间实现数据共享的功能，还可以通过 Content Provider 来实现跨进程之间的通信。在 Android 系统自带有很多 Content Provider（通信录，短信等）。
  2. 在 Content Provider 的 回调方法中， onCreate() 方法是运行在主线程（UI线程）中，在 AMS 通过 ActivityThread 创建并回调 onCreate() 方法，不能再 onCreat() 方法中耗时操作。其他方法 query(), insert(), update(), delete() 等方法都是运行在应用 程序的 Binder 维护的线程池中，所以在其他方法耗时操作不会阻塞主线程。

----
### [Dagger2](https://www.jianshu.com/p/24af4c102f62)
  - Dagger 2 是一个依赖注入框架，在编译期间自动生成代码，负责依赖对象的创建。
  - 使用 Dagger 2 的好处就是项目解耦。

---
