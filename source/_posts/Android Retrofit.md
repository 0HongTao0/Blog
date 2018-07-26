---
title: Android Retrofit
date: 2018-07-26 10:48:42
tags:





---

##### 简单介绍一下 [Retrofit](https://square.github.io/retrofit/)

- Type-safe HTTP client for Android and Java by Square ，适用于 Android 和 Java 客户端进行网络请求。

- 通过注解描述 Http 请求，Url 参数和 Request 请求参数。

- 支持自定义 Header，多类型请求体，文件上传和下载，**模仿服务器返回的 Response（Mocking Responses）**

<!--more-->

##### 项目导入 Retrofit

- Gradle 文件添加依赖**(Retrofit 1.9 以下版本需要自行添加 OKHttp 的依赖)**

  ```groovy
  implementation 'com.squareup.retrofit2:retrofit:2.4.0' //Retrofit 的依赖包
  implementation 'com.squareup.retrofit2:converter-gson:2.3.0' // Retrofit 的 Gson 转换器依赖包
  ```

- 添加网络权限

  ```xml
  <uses-permission android:name="android.permission.INTERNET"/>
  ```

##### 使用 Retrofit 进行简单的网络请求（借用 [Github 仓库接口](https://api.github.com/users/{user}/repos)）

- 定义进行网络请求的 Api 接口

  ```java
  public interface GitHubClient {
      //请求方法 GET，填入的是 url ，请求参数 user 对应接口方法 reposForUser 的 user 值
      @GET("/users/{user}/repos") 
      Call<List<GitHubRepo>> reposForUser(
              @Path("user") String user
      );
  }
  ```

- 根据数据返回类型进行定义 Model （数据返回较多，选取部分进行解析）

  ```java
  public class GitHubRepo {
      private int id;
      private String name;
  
      public GitHubRepo() {
      }
  
      public int getId() {
          return id;
      }
  
      public String getName() {
          return name;
      }
  }
  ```

- 进行网络请求（Request）和获取返回结果（Response）

  ```java
  Retrofit retrofit = new Retrofit.Builder()//// 获取 Retrofit 的构造器,通过构造器构造 Retrofit 实例
          .baseUrl(API_BASE_URL) //baseUrl 要与配置接口中的请求方法中的 url 拼接
          .addConverterFactory(GsonConverterFactory.create())//转换器是 Gson 转换器，通过 GsonConverterFactory 获得
          .client(NetWordStudyApplication.getHttpClient().build()) //使用 OKHttp 进行网络请求
          .build();
  
  GitHubClient client = retrofit.create(GitHubClient.class);//获得对应接口的网络请求
  
  Call<List<GitHubRepo>> call = client.reposForUser("Xiao");//进行网络请求，并且获取请求结果
  
  // Callback 的泛型解释：经过 Gson 转换器通过此泛型对返回数据进行转换解析。
  call.enqueue(new Callback<List<GitHubRepo>>() {
      @Override
      public void onResponse(Call<List<GitHubRepo>> call, Response<List<GitHubRepo>> response) {
          //请求成功并返回结果在此进行处理，请求数据在 response.body() 里面（对应 Callback 泛型）
          }
      }
  
      @Override
      public void onFailure(Call<List<GitHubRepo>> call, Throwable t) {
          //请求失败，在此获取请求失败的 Throwable 原因等。
      }
  });
  ```

##### 使用 Java 注解来定义 Api 接口的内容

通常，我们的 Api 接口都应该包含 3 部分：请求方法名字，返回类型，请求方法参数。

- 请求方法：Retrofit 提供 5 种 Http 请求方法，使用注解：@GET, @POST, @PUT, @DELETE, @PATCH, @HEAD。具体功能是 [Http 协议](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)内容，在此不再累述。此外，你需要添加**相对的 Api Url** 作为请求方法的参数，因为 Retrofit 提供一个设置 baseUrl 的方法（优点：方便改变全局 Url ），也可以直接定义**完整的 Api Url 路径**，或者你不用直接定义 Url，而是通过参数传入，**动态指定请求 Url** 。

- 返回类型：Call<T> 中的泛型 T ，就是 Converter（Retrofit 转换器）通过解析请求 Api 接口的返回数据成的实例。假如想获得原始数据，而不是解析出来的数据，则该泛型 T 应该设置为 ResponseBody 。或者你根本不需要知道此请求的结果，也可以将泛型 T 设置为 Void 。

- 请求方法参数：

| 请求方法参数注解 | 解释                             |
|:--------:|:------------------------------:|
| @Body    | 将 Java 对象作为请求体（Request Body）发送 |
| @Url     | 动态传入的 Url                      |
| @Filed   | 将数据作为 Form 表单发送                |

  以下是对应的一些例子：

```java
  public interface FutureStudioClient {  
      @GET("/user/info")
      Call<UserInfo> getUserInfo();

      @PUT("/user/info")
      Call<UserInfo> updateUserInfo(
          @Body UserInfo userInfo
      );

      @DELETE("/user")
      Call<Void> deleteUser();

      @GET("https://futurestud.io/tutorials/rss/") //完整的 Api Url 路径
      Call<FutureStudioRssFeed> getRssFeed();

      @GET
      Call<ResponseBody> getUserProfilePhoto(
          @Url String profilePhotoUrl //动态传入 Url
      );
  }
```

Api 的请求参数：一般来说，我们的 Api 请求方法是 GET 和 POST，GET 的请求参数是直接拼接在 Url 进行请求，而 POST 的请求参数是放在 Body 里面进行请求。

- Api Url 为 https://api.github.com/users/{user}/repos （**REST APIs**）这种占位符形式，使用 @Path 让接口方法的参数替换在 Url 中  {user} 的值。例子：

  ```java
  public interface GitHubClient {  
      @GET("/users/{user}/repos")
      Call<List<GitHubRepo>> reposForUser(
          @Path("user") String user
      );
  }
  ```

- Api Url 为：https://futurestud.io/tutorials?page=page&order=order&author=author&published_at=date ，其中 ?Xxx=xxx 就是以一种键值对的形式传入。此时可以使用 @Query 将接口方法参数拼接成这种形式的 Url 。例子：

  ```java
  public interface NewsService() {  
      @GET("/news")
      Call<List<News>> getNews(
              @Query("page") int page,
              @Query("order") String order,
              @Query("author") String author,
              @Query("published_at") Date date,
      );
  }
  ```

  当然，多个请求参数可以使用 @QueryMap，上面请求参数的表现形式等同于：

  ```java
  public interface NewsService() {  
      @GET("/news")
      Call<List<News>> getNews(
          @QueryMap Map<String, String> options
      );
  }
  
  private void fetchNews() {  
      Map<String, String> data = new HashMap<>();
      data.put("author", "Marcus");
      data.put("page", String.valueOf(2));
      //.....
      Call<List<News>> call = newsService.getNews(data);
      call.enqueue(…);
  }
  ```

##### 对 Retrofit 进行封装（ServiceGenerator）

```java
public class ServiceGenerator {
    // App 中的 BaseUrl，若更改 Url，即可在此直接修改。
    private static final String BASE_URL = "https://api.github.com/";

    private static Retrofit.Builder builder = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create());// Gson 转换器

    private static Retrofit retrofit = builder.build();
	
    private static HttpLoggingInterceptor logging = new HttpLoggingInterceptor()//自定义的登录拦截器
                    .setLevel(HttpLoggingInterceptor.Level.BODY);
					
    private static OkHttpClient.Builder httpClient =new OkHttpClient.Builder(); //使用 OKHttp 进行网络请求

      // Api 接口泛型封装，参数：带注解配置的 Api 请求接口，返回即可执行网络请求。
    public static <S> S createService(Class<S> serviceClass) {
      	if (!httpClient.interceptors().contains(logging)) {//在此对 OKHttp 添加登录拦截器的拦截
            httpClient.addInterceptor(logging);
            builder.client(httpClient.build());
            retrofit = builder.build();
        }
        return retrofit.create(serviceClass);
    }
}
```

**上面的成员变量和成员方法都是 static （静态）的，原因是整个 App 进行网络请求也只要有一个出口就行，不需要那么多实例（节约资源），static（静态）方法和变量是加载该类的时候就已经加载了的，类对应的实例只有一个该静态变量，所以性能上比较快。**

在经过封装后的 Retrofit 调用就非常简单了。之前的创建 GitHubClient 实例的例子等同于下面代码：

```java
GitHubClient client = ServiceGenerator.createService(GitHubClient.class);//带注解配置的 Api 请求接口
```

当然，上面只是一个简单的封装，在 App 中的所有网络请求都会经过 ServiceGenerator 类的 createService(Class&lt;S&gt; serviceClass) 方法，还有 Retrofit 进行网络请求的是 OKHttp ，所有我们可以在此类加入 [OKHttp 的拦截器](https://github.com/square/okhttp/wiki/Interceptors) 进行对网络请求的**请求（Request）**和**返回（Response）**进行一些判断处理（是否登录等）。


