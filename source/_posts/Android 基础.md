title: Android 基础
date: 2018-11-30 14.00
tags:

------
### [Android 内存优化](https://mp.weixin.qq.com/s/Z7oMv0IgKWNkhLon_hFakg?)
  - **RAM优化** ：降低程序在运行时使用的内存，防止由于内存不足导致程序被系统杀死。（LMK 机制：LowMemoryKill）
  - ROM 优化 : 主要是降低程序的占用空间，降低 Apk 的大小，防止偶遇 ROM 不足导致程序无法被安装。

  - 内存泄漏的解决方法
    1. [AndroidExcludedRefs](https://github.com/square/leakcanary/blob/master/leakcanary-android/src/main/java/com/squareup/leakcanary/AndroidExcludedRefs.java) 列出很多由于系统原因导致引用无法释放的例子，通过 hack 的建议去修复。
    2. 兜底回收内存，Activity 泄漏导致其引用的资源无法释放，兜底回收是指将 Activity 所持有的资源都回收掉（引用置 null），然后剩下的 Activity 就是个空壳。
  - 降低运行时内存大小
    1. 减少 bitmap 占用的内存，图片按需求的 View 大小加载。统一使用 bitmap 的加载器，在发生 OOM 时清除缓存。
    2. 监控程序堆内存的使用率，达到程序的设定值则启动相关模块进行内存释放。
    3. 多进程：将对于会引发内存泄漏或者占用内存过大的组件放置单独的进程中运行。
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
