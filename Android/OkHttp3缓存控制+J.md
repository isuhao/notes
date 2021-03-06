
# OkHttp3缓存控制

> 也适用于Retrofit，最初也是来源于**Retrofit设置缓存**  

> baseUrl + EndPoint(端点) = 组合成 url  ？？？

## 相关概念   


浏览器缓存控制机制有两种：**HTML Meta标签** vs. **HTTP头信息**  
浏览器缓存机制，其实主要就是HTTP协议定义的缓存机制（如： `Expires`； `Cache-control`等）。但是也有非HTTP协议定义的缓存机制，如使用HTML Meta 标签，Web开发者可以在HTML页面的`<head>`节点中加入`<meta>`标签，但只有部分浏览器可以支持。

## 几个重要概念解释    

**Expires策略：**Expires是Web服务器响应消息头字段，在响应http请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，而无需再次请求。不过Expires 是HTTP 1.0的东西，现在默认浏览器均默认使用HTTP 1.1，所以它的作用基本忽略。   
**Cache-control策略（重点关注）：**Cache-Control与Expires的作用一致，都是指明当前资源的有效期，控制浏览器是否直接从浏览器缓存取数据还是重新发请求到服务器取数据。只不过Cache-Control的选择更多，设置更细致，如果同时设置的话，其优先级高于Expires。

1.用在request中的cache控制头:   

> Pragma: no-cache :兼容早起HTTP协议版本 如1.0+   
> Cache-Control: no-cache ，表示不希望得到一个缓存内容。只是希望，cache设备可能忽略。   
> Cache-Control: no-store，表示client与server之间的设备不能缓存响应内容，并应该删除已有缓存。   
> Cache-Control: only-if-cached，表示只接受是被缓存的内容  

2.用在response中控制cache的头:  

> Cache-Control: max-age=3600，用相对于接收到的时间开始可缓存多久
> Cache-Control: s-maxage=3600，与上面类似，只是s-maxage一般用在cache服务器上，并只对public缓存有效
> Expires: Fri, 05 Jul 2002, 05:00:00 GMT基于GMT的时间，绝对时间，但该头容易受到本地错误时间影响
> Cache-Control: must-revalidate 该头表示内容可以被缓存但每次必须询问是否有更新。

3.常规标头 : (通用标头)在请求和响应中都可以使用的一些常规标头,如HTTP1.0所定义的**Pragma**(是的,这并不是一个实体标头,它仅仅是试图与客户端,包括代理,进行协商,不要使用缓存的副本，在后面设置缓存时我们一般会在拦截器中移除 Pragma).




4.cache-control的缓存指令：

> 缓存指令可以是 public、private、no-cache、no- store、no-transform、must-revalidate、proxy-revalidate、max-age  
>
> 各个消息中的指令含义如下：    
> Public: 指示响应可被任何缓存区缓存。    
> Private: 指示对于单个用户的整个或部分响应消息，不能被共享缓存处理。这允许服务器仅仅描述当用户的部分响应消息，此响应消息对于其他用户的请求无效。    
> no-cache: 指示请求或响应消息不能缓存，该选项并不是说可以设置”不缓存“，容易望文生义   
> no-store: 用于防止重要的信息被无意的发布。在请求消息中发送将使得请求和响应消息都不使用缓存，完全不存下來。   
> max-age: 指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。   
> min-fresh: 指示客户机可以接收响应时间小于当前时间加上指定时间的响应。   
> max-stale: 指示客户机可以接收超出超时期间的响应消息。如果指定max-stale消息的值，那么客户机可以接收超出超时期指定值之内的响应消息。 

Last-Modified/If-Modified-Since要配合Cache-Control使用：     
略


Etag/If-None-Match也要配合Cache-Control使用：    
略

> 参考：[Retrofit2.0+Okhttp不依赖服务端的数据缓存](http://dandanlove.com/2016/09/18/retrofit-okhttp-cache-offline/)  
[浏览器 HTTP 协议缓存机制详解](http://blog.csdn.net/stven_king/article/details/51899865)     
[你应该了解的 一些web缓存相关的概念](http://www.cnblogs.com/_franky/archive/2011/11/23/2260109.html "你应该了解的 一些web缓存相关的概念. - Franky - 博客园")      
[高效地配置OkHttp - 开发技术前线](http://www.devtf.cn/?p=1264 "高效地配置OkHttp - 开发技术前线")  



## 基本思路

1. 配置Okhttp的Cache，设置最大容量和缓存路径。当缓存的大小超过最大容量时，OKHttp会根据LRU算法对缓存数据进行清理。
2. 配置**请求头中**的`cache-control`或者统一处理所有请求的请求头。
3. 云端配合设置响应头或者自己写拦截器修改响应头中`cache-control`。

4. 我们所用的接口服务不支持缓存，所以我不能只修改头信息而让服务端返回的response响应体去实现数据本地缓存。当然在没有网络的情况下我们可以尝试去读取缓存。
5. 因为服务端没有提供response响应体的缓存，所以我们清除response响应体的Pragma、Cache-Control信息，**然后根据自己设定的request请求体中的Cache信息去修改response响应体的Cache信息从而达到数据可以缓存**。
6. 在开发的过程中遇到如果一个接口在某次请求返回404，那么以后的结果总是请求失败的404页面。所以在请求失败的时候需要初始化OkHttpClient实例。


## 实现步骤

[使用Retrofit和Okhttp实现网络缓存。无网读缓存，有网根据过期时间重新请求 - 简书](http://www.jianshu.com/p/9c3b4ea108a7 "使用Retrofit和Okhttp实现网络缓存。无网读缓存，有网根据过期时间重新请求 - 简书")



1. 配置okhttp中的Cache。
2. 配置**请求头**中的cache-control，在Retrofit中，我们可以通过`@Headers`来配置（在需要缓存的接口上单独进行设置，这种方法能让每个接口的缓存时间都不同）添加如下注解`@Headers("Cache-Control: public, max-age=3600")`。也可在拦截器中设置（通过拦截Request，并添加更改的方式。这就是为什么有些文章没有写通过`@Headers`来配置的原因）
3. 云端配合设置响应头或者**自己写拦截器修改响应头response中cache-control**
4. 设置拦截器



> max-stale在请求头设置有效，在响应头设置无效。因为max-stale是请求头设置参数。
> max-stale和max-age同时设置的时候，缓存失效的时间按最长的算。


**设置拦截器：**  
REWRITE_CACHE_CONTROL_INTERCEPTOR拦截器需要同时设置networkInterceptors和interceptors（OKHTTP3.0配置是否有效待我测试）



## 补充



### 缓存目录



[Android 缓存目录 Context.getExternalFilesDir()和Context.getExternalCacheDir()方法 - 赵彦军 - 博客园](http://www.cnblogs.com/zhaoyanjun/p/4530155.html "Android 缓存目录 Context.getExternalFilesDir()和Context.getExternalCacheDir()方法 - 赵彦军 - 博客园")

Jesse Wilson大神推荐，我们把缓存的数据存放在 `context.getCacheDir()`的子目录中。

```java
String cachePath1 =  context.getExternalCacheDir().getPath();
String cachePath2 =  context.getCacheDir().getPath();
KLog.d( "cachePath1--" + cachePath1 ) ;
KLog.d( "cachePath2--" + cachePath2 )
```

结果：

```
cachePath1--/storage/emulated/0/Android/data/com.zyj.app/cache
cachePath2--/data/user/0/com.zyj.app/cache
```



> `/storage/emulated/0/` 这个目录比较常见哦，比如当我使用File Manager + 查看文件的属性时，就有这段路径。  
>
> 而`/`根目录下的文件需要，开启显示系统文件才能看见，但是如果手机没有root，只能列出根目录下的文件夹，而不能进入查看文件。



`context.getExternalCacheDir()  、 context.getCacheDir()`

相同点：

> 1、相同点：都可以做app缓存目录。
>
> 2、app卸载后，两个目录下的数据都会被清空。

不同点：

> 1、目录的路径不同。前者的目录存在外部SD卡上的。后者的目录存在app的内部存储上。
>
> 2、前者的路径在手机里可以直接看到。后者的路径需要root以后，用Root Explorer 文件管理器才能看到。

注意：  

由于context.getExternalCacheDir() 的目录存在外部SD卡上的，所以在使用这个方法的时候要判断外部SD卡的状态是否可用。

由于Android 版本的升级，该方法可能无法正常检测，请上网搜索最新的相关文章。



为了使您的代码更加健壮并且能够兼容以后的[android](http://lib.csdn.net/base/android)版本和新的设备，请通过Environment.getExternalStorageDirectory().getPath()来获取sdcard路径。

通用的获取语句

```
Environment.getExternalStorageDirectory().getPath()  
```
当外置sd卡不存在的情况下，这条语句是获取的内置sd卡的路径。外置sd卡存在，获取是外置sd卡。获取出来的值是`/storage/sdcard0`

```java
/**
  * 获取app缓存路径
  * @param context
  * @return
  */
 public String getCachePath( Context context ){
     String cachePath ;
     if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
             || !Environment.isExternalStorageRemovable()) {
         //外部存储可用
         cachePath = context.getExternalCacheDir().getPath() ;
     }else {
         //外部存储不可用
         cachePath = context.getCacheDir().getPath() ;
     }
     return cachePath ;
 }
```



getExternalCacheDir()方法使用的目录还对应 `设置->应用->应用详情`里面的`”清除缓存“`选项。



### Cache-Control

`"Cache-Control: public, max-age=3600"` 中，时间单位为秒，`3600  = 60秒 × 60分钟 = 1小时`

```
Common max-age values are:

One minute: max-age=60
One hour: max-age=3600
One day: max-age=86400
One week: max-age=604800
One month: max-age=2628000
One year: max-age=31536000
```

> 以下是给webmaster(网站管理员)的建议： 

>**File types**

>Two questions a webmaster(网站管理员) should ask are:
>
  - What file types should I cache?
  - How long should I cache them for?

> 列举的文件：  
>
- Images - (png, jpg, gif, etc.) Images don't really change so they can have a long cache time period (a year)
- CSS - CSS files tend to change more often than other files, a shorter time period may be needed (week or month)
- ico (favicon) - rarely changed (a year)
- JS - Javascripts for the most part are not changing very often so a longer cache time is normally possible (a month)

> 缓存指令Caching Directives：  
  - public : anyone can cache this at any level.
  - private ： The private directive means it is specific to one person.An example would be a Twitter page. When you go to Twitter you see one thing and another person going to the same url sees something else。表示响应消息是针对单个用户而不是由共享缓存存储。
  - no-store ： 



对应的CacheControl类

```java
final CacheControl.Builder builder = new CacheControl.Builder();
            builder.noCache();//不使用缓存，全部走网络
            builder.noStore();//不使用缓存，也不存储缓存
            builder.onlyIfCached();//只使用缓存
            builder.noTransform();//禁止转码
            builder.maxAge(10, TimeUnit.MILLISECONDS);//指示客户机可以接收生存期不大于指定时间的响应。
            builder.maxStale(10, TimeUnit.SECONDS);//指示客户机可以接收超出超时期间的响应消息
            builder.minFresh(10, TimeUnit.SECONDS);//指示客户机可以接收响应时间小于当前时间加上指定时间的响应。
            CacheControl cache = builder.build();//cacheContro
```

两个CacheControl常量介绍：
```java
CacheControl.FORCE_CACHE; //仅仅使用缓存
CacheControl.FORCE_NETWORK;// 仅仅使用网络
```


## Interceptor拦截器

> Servlet中有Filter过滤器

以下内容来自官方Wiki

[Interceptors · square/okhttp Wiki](https://github.com/square/okhttp/wiki/Interceptors "Interceptors · square/okhttp Wiki")

Interceptors can be chained. 拦截器可以进行链接。OkHttp使用列表来跟踪拦截器，拦截器按顺序调用。

调用 `chain.proceed(request)`是每个拦截器实现的关键部分；这个简单的方法是所有HTTP工作发生的地方，产生满足请求的响应。

Interceptors are registered as either *application* or *network* interceptors.   
okhttp的拦截器分为两种：  

- Application Interceptors

  Register an *application* interceptor by calling `addInterceptor()` on `OkHttpClient.Builder`

- Network Interceptors

  Registering a network interceptor is quite similar. Call `addNetworkInterceptor()` instead of `addInterceptor()`:




**Application interceptors**

- Don't need to worry about intermediate responses like redirects and retries.
- Are always invoked once, even if the HTTP response is served from the cache.
- Observe the application's original intent. Unconcerned with OkHttp-injected headers like `If-None-Match`.
- Permitted to short-circuit and not call `Chain.proceed()`.
- Permitted to retry and make multiple calls to `Chain.proceed()`.

**Network Interceptors**

- Able to operate on intermediate responses like redirects and retries.
- Not invoked for cached responses that short-circuit the network.
- Observe the data just as it will be transmitted over the network.
- Access to the `Connection` that carries the request.

Network Interceptors让所有网络请求都附上你的拦截器，包括 redirects and retries


### Rewriting Requests  重写请求

Interceptors can add, remove, or replace request headers. They can also transform the body of those requests that have one. For example, you can use an application interceptor to add request body compression if you're connecting to a webserver known to support it.


```java
final class GzipRequestInterceptor implements Interceptor {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    //获取原始请求
    Request originalRequest = chain.request();
    if (originalRequest.body() == null || originalRequest.header("Content-Encoding") != null) {
     //产生满足该请求的响应
      return chain.proceed(originalRequest);
    }

	//在原请求的基础上newBuilder一个请求
    Request compressedRequest = originalRequest.newBuilder()
        .header("Content-Encoding", "gzip")
        .method(originalRequest.method(), gzip(originalRequest.body()))
        .build();
    ////产生满足该请求的响应
    return chain.proceed(compressedRequest);
  }

  private RequestBody gzip(final RequestBody body) {
    return new RequestBody() {
      @Override public MediaType contentType() {
        return body.contentType();
      }

      @Override public long contentLength() {
        return -1; // We don't know the compressed length in advance!
      }

      @Override public void writeTo(BufferedSink sink) throws IOException {
        BufferedSink gzipSink = Okio.buffer(new GzipSink(sink));
        body.writeTo(gzipSink);
        gzipSink.close();
      }
    };
  }
}
```



### Rewriting Responses 重写响应

对称地，拦截器可以重写响应头并转换响应体。您可以修复服务器配置错误的Cache-Control响应头以实现更好的响应缓存

```java
/** Dangerous interceptor that rewrites the server's cache-control header. */
//REWRITE_CACHE_CONTROL_INTERCEPTOR：意思是，重写缓存控制拦截器
private static final Interceptor REWRITE_CACHE_CONTROL_INTERCEPTOR = new Interceptor() {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    //获取原始响应
    Response originalResponse = chain.proceed(chain.request());
    //在原始响应的基础上newBuilder一个响应
    return originalResponse.newBuilder()
        .header("Cache-Control", "max-age=60")
        .build();
  }
};
```

> 可以在同一个拦截器中同时进行重写请求和重写响应。很多示例都是这样做的。


### 日志拦截器 LoggingInterceptor 

用法：自定义日志拦截器(自定义日志打印的格式和信息)
```java
// chain -> 这个语法为Lambda表达式，chain代表传入方法的参数
private final Interceptor LoggingInterceptor = chain -> { 
    Request request = chain.request(); 
    long t1 = System.nanoTime();
    Logger.t(TAG).i(String.format("Sending request %s on %s%n%s", request.url(),  chain.connection(), request.headers()));
    Response response = chain.proceed(request); 
    long t2 = System.nanoTime(); 
    Logger.t(TAG).i(String.format("Received response for %s in %.1fms%n%s", response.request().url(), (t2 - t1) / 1e6d, response.headers())); 
    return response; 
};
```

然后向其他拦截器一样进行添加即可。（需要最后添加日志拦截器，因为拦截器是按顺序执行的）



使用Retrofit2时，如果导入了如下从okHttp3中分离出的的包，还可以使用如下自带的日志过滤器：  

The developers of OkHttp have released a separate(分离出一个单独的) logging interceptor project, which implements logging for OkHttp.

```
compile 'com.squareup.okhttp3:logging-interceptor:3.8.0'
```

然后就可以使用如下代码使用相关过滤器

```java
HttpLoggingInterceptor logging = new HttpLoggingInterceptor();
logging.setLevel(HttpLoggingInterceptor.Level.BODY);
// add your other interceptors …

// add logging as last interceptor（该拦截器在最后添加）
httpClient.addInterceptor(logging);   // <-- this is the important line!
```

[HttpLoggingInterceptor (OkHttp Logging Interceptor 3.8.0 API)](https://square.github.io/okhttp/3.x/logging-interceptor/okhttp3/logging/HttpLoggingInterceptor.html "HttpLoggingInterceptor (OkHttp Logging Interceptor 3.8.0 API)")    
[Retrofit 2 — Log Requests and Responses](https://futurestud.io/tutorials/retrofit-2-log-requests-and-responses "Retrofit 2 — Log Requests and Responses")





## 用户认证

OkHttp 提供了对用户认证的支持。当 HTTP 响应的状态代码是 401 时，OkHttp 会从设置的 Authenticator 对象中获取到新的 Request 对象并再次尝试发出请求。Authenticator 接口中的 authenticate 方法用来提供进行认证的 Request 对象.

```java
OkHttpClient client = new OkHttpClient();
        client.newBuilder().authenticator(new Authenticator() {
            @Override
            public Request authenticate(Route route, Response response) throws IOException {
                String credential = Credentials.basic("user", "password");
                return response.request().newBuilder()
                        .header("Authorization", credential)
                        .build();
            }
        });
```


让所有网络请求都附上你的 token



## Retrofit源码

Retrofit源码，中http包里面的几个类和接口：


![](http://img.blog.csdn.net/20160512174753189)

**Call:**        
这个接口主要的作用就是发送一个Http请求，Retrofit的默认请求方式是OKHttpCall,当然你也可以根据自己的业务逻辑自己定义Call。

**CallAdapter：**        
这个接口的主要作用就是将Call对象转化成另一个对象，原谅我的水平有限，没太看懂里面的代码

**CallBack：**         
看接口名想必大家都能看出来，这是回掉接口，里面有两个回调方法 
onResponse()       
onFailure().      

**Converter:**       
这个接口主要的作用是将服务器返回数据解析为你所需要的JSON，XML等对象。

**OkHttpCall：**       
OkHttpCall实现了上面的Call接口，通过这个类直接用OkHttp的request去执行网络请求，实现异步，同步请求，接口回调…

**ServiceMethod:**       
这个类主要是用来通过解析注解、传参，将它们封装成Request，然后通过具体的返回值类型，让我们自己配置的工厂生成具体的CallAdapter。       

