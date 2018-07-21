---

title: Android Volley

date: 2018-07-19 09:27:42

tags:

---

##### 简单介绍一下 Volley

- Volley 是一个 Http 协议的 Android 网络框架，它可以自动调度 App 的网络请求，可以并发执行网络请求，具有自己的一套网络请求的响应缓存（和 HTTP 的缓存不一样）。Volley 还支持设置优先级的网络请求，中断正在执行的网络请求。**Volley 支持开发者自定义网络请求返回的类型。**总的来说 Volley 就是一个异步网络请求的框架。<!--more-->

##### 如何在 Android 项目中引入 Volley

**框架比较老，并且没有更新了，所以强烈建议使用方法二。jar 包可以搜索一下就能找到**

- 方法一：

  直接在 Android 的依赖 Gradle 文件中添加依赖（但是不知道为什么总是下载不下来）

  ```groovy
  dependencies {
      implementation 'com.android.volley:volley:1.1.1'
  }
  ```

- 方法二：

  下载 Volley jar 包，添加到项目的 libs 目录下，并添加 Jar dependency 

  ```groovy
  dependencies {
      implementation files('libs/volley.jar')
  }
  ```

- 方法三：

  去 Volley 的 Github 仓库 clone 源码，并在 Android 的项目中进行 import module 

  ```
  git clone https://github.com/google/volley
  ```

##### 使用 Volley 实现一个简单的网络请求

```java
private static final String TAG = MainActivity.class.getSimpleName();
/**
 * 网络请求的队列(通常在 Application 类中初始化，并全局唯一一个 RequestQueue)
 */
private RequestQueue mRequestQueue;
private static final String URL = "http://www.awqingnian.xyz/2018/07/18/Android%20View/"

//初始化 RequestQueue
mRequestQueue = Volley.newRequestQueue(this);
//执行一个返回 String 的网络请求
StringRequest request = new StringRequest(Request.Method.GET, URL,
        new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                Log.d(TAG, "onResponse: " + response);
            }
        }, new Response.ErrorListener() {
    @Override
    public void onErrorResponse(VolleyError error) {
        Log.d(TAG, "onErrorResponse: 网络请求错误   " + error);
    }
});

// 将网络请求添加到请求队列中（队列自动执行网络请求）
mRequestQueue.add(request);
```

当然上面举的例子是简单的例子，Volley 的网络请求不仅仅只有返回 String 类型的，还有其他很多类型。

| Request           | 返回类型                            |
|:-----------------:|:-------------------------------:|
| ImageRequest      | 请求指定 image 的 Url 返回一个解码的 Bitmap |
| ClearCacheRequest | 用于清除 Volley 网络请求缓存              |
| StringRequest     | 请求指定 Url 返回 String 类型的值         |
| JsonRequest       | 请求指定 Url 返回 JSON 值              |
| JsonObjectRequest | 请求指定 Url 返回 JsonObject          |
| JsonArrayRequest  | 请求指定 Url 返回 JsonArray           |

##### 自定义 Request （高级用法）

- 主要是通过请求 Url 返回的 json，经过 Gson 解析，以及 java 泛型的传入的 Class<T>，返回 T 类所对应的对象。当然开发者可以自定义很多类型的返回类型，怎么自定义，参考 Request 源码的写法加上自己的思考就能实现。下面介绍自定义 Request ，返回 List 集合：

  ```java
  /**
   * 继承 Request ，并将泛型 <T> 传入
   */
  public class DiyGSONRequest<T> extends Request<T> {
      private final Response.Listener<List<T>> mListener;
      private Class<T> mClass;
      //5 参数构造方法
      private DiyGSONRequest(int method, String url, Class<T> clazz, Response.Listener<List<T>> listener, Response.ErrorListener errorListener) {
          super(method, url, errorListener);
          mClass = clazz;
          mListener = listener;
      }
      //4 参数构造方法    （默认请求类型 GET）
      public DiyGSONRequest(String url, Class<T> clazz, Response.Listener<List<T>> listener, Response.ErrorListener errorListener) {
          this(Method.GET, url, clazz, listener, errorListener);
      }
      //请求成功后，对 networkResponse 进行解析成泛型 T 的对象
      @Override
      protected Response<T> parseNetworkResponse(NetworkResponse networkResponse) {
          String jsonData;
  
          try {
              jsonData = new String(networkResponse.data, HttpHeaderParser.parseCharset(networkResponse.headers));
              List<T> list = new ArrayList<>();
              JsonArray array = new JsonParser().parse(jsonData).getAsJsonArray();
              for (JsonElement element : array) {
                  // Gson 解析
                  list.add(new Gson().fromJson(element, mClass));
              }
              return (Response<T>) Response.success(list, HttpHeaderParser.parseCacheHeaders(networkResponse));
          } catch (UnsupportedEncodingException e) {
              return Response.error(new ParseError(e));
          }
      }
  
      @Override
      protected void deliverResponse(T t) {
          mListener.onResponse((List<T>) t);
      }
  }
  ```

  ```
  //数据类型可以使用这组数据测试
  [{"id":1,"name":"北京"},{"id":2,"name":"上海"},{"id":3,"name":"天津"},{"id":4,"name":"重庆"}]
  ```

##### 简单介绍 Volley 的几个重要的类

- Request （网络请求的基类）

  构造方法如下：

  ```java
  public Request(String url, Response.ErrorListener listener) {
          this(Method.DEPRECATED_GET_OR_POST, url, listener);
  }
  
  /**
   *  method: 请求类型（GET : 0, POST : 1, PUT : 2, DELETE : 3）
   *  url: 请求地址的 Url
   *  Response.ErrorListener : 请求错误的回调接口
   */
  public Request(int method, String url, Response.ErrorListener listener) {
	  mMethod = method;
	  mUrl = url;
	  mErrorListener = listener;
	  setRetryPolicy(new DefaultRetryPolicy());

	  mDefaultTrafficStatsTag = TextUtils.isEmpty(url) ? 0: Uri.parse(url).getHost().hashCode();
  }
  
  /**
   * Request 子类必须实现的抽象方法，当执行该网络请求成功后，将网络请求回调到此接口方法
   */
  abstract protected void deliverResponse(T response);
  ```

- Volley（用来实例化 RequestQueue 实例）

  获取 RequestQueue 实例的方法在 Volley 类中统一管理。

  ```java
  public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
    // 配置缓存的路径文件（Volley 的缓存模块）
    File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);
  
    String userAgent = "volley/0";
    try {
      String packageName = context.getPackageName();
      PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
      userAgent = packageName + "/" + info.versionCode;
    } catch (NameNotFoundException e) {
    }
  
    if (stack == null) {
      if (Build.VERSION.SDK_INT >= 9) {
        //现在手机 SDK 基本都在 19 以上，所以只要关注这块就行了。
        //HurlStack 就是一个底层封装了 HttpURLConnection 的类，主要用来执行网络请求。
        stack = new HurlStack();
      } else {
        stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
      }
    }
      // BasicNetwork 是一个获取 HurlStack 执行完网络请求获取 Response 的类
    Network network = new BasicNetwork(stack);
      // RequestQueue 默认有缓存，在此初始化 RequestQueue
    RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
    //在此处开始对 RequestQueue 的 CacheDispatcher（执行缓存线程） 和 NetworkDispatcher（执行网络请求线程） 进行初始化并且开启线程，不断地从 RequestQueue 中取出 request 执行。
    queue.start();
  
    return queue;
  }
  ```

- Cache （进行对网络请求缓存的操作类）

  Cache 接口的具体实现类：DiskBasedCache

| 重要方法或变量                      | 含义                                 |
|:----------------------------:|:----------------------------------:|
| mTotalSize                   | 已经使用的缓存空间                          |
| mRootDirectory               | 缓存所在的根目录（默认："volley"）              |
| mMaxCacheSizeInBytes         | 最大缓存空间大小（默认：5 \* 1024 \* 1024（5M）） |
| clear()                      | 清除缓存                               |
| get(String key)              | 获取缓存实例（E）                          |
| initialize()                 | 初始化缓存，并将读取所有缓存存在集合中（mEntries）      |
| put(String key, Entry entry) | 存入缓存                               |
| remove(String key)           | 删除指定 key 值的缓存                      |

其中 Entry 是缓存操作的类。

- RequestQueue（网络请求的队列）

  主要方法如下

  ```java
  public RequestQueue(Cache cache, Network network, int threadPoolSize,ResponseDelivery delivery) {
	// 之前传入的 DiskBasedCache 实例，用于缓存
	mCache = cache;
	// 用于使用 HttpUrlConnection 执行网络请求并且得到 Response
	mNetwork = network;
	// 执行网络请求的线程数量（默认 4 个）
	mDispatchers = new NetworkDispatcher[threadPoolSize];
	mDelivery = delivery;
  }

  public RequestQueue(Cache cache, Network network, int threadPoolSize) {
	// 注意注意注意： 此处的 ExecutorDelivery 是通过主线程的 Looper 创建的，也就是说此 ExecutorDelivery 执行在主线程中（异步请求的重点）
	this(cache, network, threadPoolSize,new ExecutorDelivery(new Handler(Looper.getMainLooper())));
  }

  public Request add(Request request) {

	//······省略一些代码······

	// 不需要缓存，直接加入队列中
	if (!request.shouldCache()) {
	  mNetworkQueue.add(request);
	  return request;
	}

	synchronized (mWaitingRequests) {
	  String cacheKey = request.getCacheKey();
	  if (mWaitingRequests.containsKey(cacheKey)) {
		//等待队列存在此网络请求
		Queue<Request> stagedRequests = mWaitingRequests.get(cacheKey);
		if (stagedRequests == null) {
		  stagedRequests = new LinkedList<Request>();
		}
		stagedRequests.add(request);
		mWaitingRequests.put(cacheKey, stagedRequests);
		if (VolleyLog.DEBUG) {
		  VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);
		}
	  } else {
		//等待队列不存在此网络请求，加入等待队列中
		mWaitingRequests.put(cacheKey, null);
	  }
	  return request;
  }
```

	
- NetworkDispatcher（extends Thread）

  run 方法执行网络请求并异步回调结果的代码如下

  ```java
  public void run() {
    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
    Request request;
    while (true) {

      //······省略一些代码······

      try {
        request.addMarker("network-queue-take");
        if (request.isCanceled()) {
          request.finish("network-discard-cancelled");
          continue;
        }
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
          TrafficStats.setThreadStatsTag(request.getTrafficStatsTag());
        }
        //在此处通过 NetWork 进行网络请求并且的搭配返回的 NetworkResponse
        NetworkResponse networkResponse = mNetwork.performRequest(request);
        request.addMarker("network-http-complete");

        //······省略一些代码······

        //在此进行对网络请求进行缓存
        if (request.shouldCache() && response.cacheEntry != null) {
          mCache.put(request.getCacheKey(), response.cacheEntry);
          request.addMarker("network-cache-written");
        }
        request.markDelivered();
        // 注意注意注意：在此通过 ExecutorDelivery（执行在主线程中）将 response 异步回调到主线程的 request 回调函数中处理结果
        mDelivery.postResponse(request, response);
      } catch (VolleyError volleyError) {
        parseAndDeliverNetworkError(request, volleyError);
      } catch (Exception e) {
        VolleyLog.e(e, "Unhandled exception %s", e.toString());
        mDelivery.postError(request, new VolleyError(e));
      }
    }
  }
```

- ResponseDelivery  (对在 NetworkDispatcher 网络请求返回的 Response 进行通过 Request 的deliverResponse(T response) )ResponseDelivery 接口的实现类：ExecutorDelivery
