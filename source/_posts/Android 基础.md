title: Android 基础
date: 2018-11-30 14.00
tags:

------
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
