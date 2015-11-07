title: Android Volley自定义Request(JacksonRequest、GsonRequest)
date: 2014-12-05 22:55:31
category: Android
tags: Volley
toc: true
---

# 前言
Volley是一个强大的HTTP库，让Android的的网络操作变得容易、高效、快速。在少数据的高并发情况下它的优势很明显，但在大文件的上传或者下载操作中就并不适用了，这种情况下是建议使用传统方式或者其它框架来实现的。
*它的优点再罗列如下：*
- 自动管理网络请求
- 多并发的网络连接
- 通过标准的HTTP cache coherence(高速缓存一致性)使得磁盘与内存缓存不可见(Transparent)
- 支持指定请求的优先级
- 支持取消已经发出的请求。你可以取消单个请求，也可以设置请求取消的块或范围
- 框架是容易被定制的，例如，重试或者回退功能
- 强大的指令使得你可以容易地去异步网络操作和(网络获取数据后)的UI设置
- (内置)Debugging和tracing工具

# 自定义Request实现
上面提到的优点：Volley是容易被定制的，它的易扩展性在这里的自定义Request实现中就很好的体现了出来。自定义Request是通过继承Volley中最核心的Request类来实现的。在我们的Request来解析json数据时，注意处理复杂json数据的解析可扩展性。比如，我们一般都需要把JSON数据解析成我们的实体对象，如下面使用示例中的Weather对象，如果我们的实体比较复杂时就需要自定义自己的Type去给Jackson、Gson去处理了，不然一般都会默认的给解析成HashMap式的键值对造成FC或者get不到属性。对于Jackson，使用Jackson库里的`TypeReference`类来实现；对于Gson，使用Gson库中的`TypeToken`类来实现。

<!--more-->

## JacksonRequest  

``` java
public class JacksonRequest<T> extends Request<T> {

	private final Listener<T> mListener;
	private static ObjectMapper objectMapper = new ObjectMapper();
	private Class<T> mClass;
	private TypeReference<T> mTypeReference;//提供和解析自定义的复杂JSON数据支持
	
	public JacksonRequest(int method, String url, Class<T> clazz, Listener<T> listener,
			ErrorListener errorListener) {
		super(method, url, errorListener);
		mClass = clazz;
		mListener = listener;
	}
	
	public JacksonRequest(int method, String url, TypeReference<T> typeReference, Listener<T> listener,
			ErrorListener errorListener) {
		super(method, url, errorListener);
		mTypeReference = typeReference;
		mListener = listener;
	}
	
	public JacksonRequest(String url, Class<T> clazz, Listener<T> listener, ErrorListener errorListener) {
		this(Method.GET, url, clazz, listener, errorListener);
	}
	
	public JacksonRequest(String url, TypeReference<T> typeReference, Listener<T> listener,
			ErrorListener errorListener) {
		super(Method.GET, url, errorListener);
		mTypeReference = typeReference;
		mListener = listener;
	}
	
	@Override
	protected Response<T> parseNetworkResponse(NetworkResponse response) {
		try {
			String jsonString = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
			if (mTypeReference == null)//使用Jackson默认的方式解析到mClass类对象，Jackson会像HashMap那样解析
				return (Response<T>) Response.success(
						objectMapper.readValue(jsonString, TypeFactory.rawClass(mClass)),
						HttpHeaderParser.parseCacheHeaders(response));
			else//通过自己构造TypeReference让Jackson解析成自定义的对象类型
				return (Response<T>) Response.success(objectMapper.readValue(jsonString, mTypeReference),
						HttpHeaderParser.parseCacheHeaders(response));
		} catch (Exception e) {
			return Response.error(new ParseError(e));
		}
	}
	
	@Override
	protected void deliverResponse(T response) {
		mListener.onResponse(response);
	}
}
```

## GonRequest

``` java
public class GsonRequest<T> extends Request<T> {

	private final Listener<T> mListener;
	private static Gson mGson = new Gson();
	private Class<T> mClass;
	private TypeToken<T> mTypeToken;//提供和解析自定义的复杂JSON数据支持,这点与Jackson使用TypeReference不同，但原理是大同小异的
	
	public GsonRequest(int method, String url, Class<T> clazz, Listener<T> listener,
			ErrorListener errorListener) {
		super(method, url, errorListener);
		mClass = clazz;
		mListener = listener;
	}
	
	public GsonRequest(int method, String url, TypeToken<T> typeToken, Listener<T> listener,
			ErrorListener errorListener) {
		super(method, url, errorListener);
		mTypeToken = typeToken;
		mListener = listener;
	}
	
	public GsonRequest(String url, Class<T> clazz, Listener<T> listener, ErrorListener errorListener) {
		this(Method.GET, url, clazz, listener, errorListener);
	}
	
	public GsonRequest(String url, TypeToken<T> typeToken, Listener<T> listener, ErrorListener errorListener) {
		super(Method.GET, url, errorListener);
		mTypeToken = typeToken;
		mListener = listener;
	}
	
	@Override
	protected Response<T> parseNetworkResponse(NetworkResponse response) {
		try {
			String jsonString = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
			if (mTypeToken == null)//与Jackson类似
				return Response.success(mGson.fromJson(jsonString, mClass),
						HttpHeaderParser.parseCacheHeaders(response));
			else
				return (Response<T>) Response.success(mGson.fromJson(jsonString, mTypeToken.getType()),
						HttpHeaderParser.parseCacheHeaders(response));
		} catch (UnsupportedEncodingException e) {
			return Response.error(new ParseError(e));
		}
	}
	
	@Override
	protected void deliverResponse(T response) {
		mListener.onResponse(response);
	}
}
``` 
从上面可以看出，通过自定义Volley的Request可以很方便的设计出符合自己指定数据类型的请求类，对于一般只关心返回结果的我们直接重写`parseNetworkResponse()`方法处理返回的结果即可，如果对`Header`什么的也需要的话就重写`getHeader()`去处理。此外我写了个Volley的请求封装[VolleyRequest](https://github.com/zhengxiaopeng/AndroidUtils/blob/master/AndroidUtils/src/com/roc/http/volley/VolleyRequest.java),代码有些长不帖上来了，大家可以点击链接看一下。现在我们可以方便的如下使用自定义的Request了，更多的用法参看[HttpVolleyTestActivity](https://github.com/zhengxiaopeng/AndroidUtils/blob/master/AndroidUtils/src/com/roc/http/volley/HttpVolleyTestActivity.java)。  

``` java
/* GsonRequest */
VolleyRequest.getInstance().newGsonRequest("http://www.weather.com.cn/data/sk/101010100.html",
        Weather.class, new Response.Listener<Weather>()
{
    @Override
    public void onResponse(Weather weather)
    {
        WeatherInfo weatherInfo = weather.getWeatherinfo();
        DebugLog.v(">>>GsonRequest: ");
        DebugLog.v("city is " + weatherInfo.getCity());
        DebugLog.v("temp is " + weatherInfo.getTemp());
        DebugLog.v("time is " + weatherInfo.getTime());
    }
}, new Response.ErrorListener()
{
    @Override
    public void onErrorResponse(VolleyError error)
    {
        DebugLog.e("GsonRequest: " + error);
    }
});

/* JacksonRequest */
VolleyRequest.getInstance().newJacksonRequest("http://www.weather.com.cn/data/sk/101010100.html",
        Weather.class, new Response.Listener<Weather>()
{
    @Override
    public void onResponse(Weather weather)
    {
        WeatherInfo weatherInfo = weather.getWeatherinfo();
        DebugLog.v(">>>JacksonRequest: ");
        DebugLog.v("city is " + weatherInfo.getCity());
        DebugLog.v("temp is " + weatherInfo.getTemp());
        DebugLog.v("time is " + weatherInfo.getTime());
    }
}, new Response.ErrorListener()
{
    @Override
    public void onErrorResponse(VolleyError error)
    {
        DebugLog.e("JacksonRequest: " + error);
    }
});
}
```

如果你的JSON需要使用自定义Type来去解析，那么你可以直接new对应的Type匿名内部类穿进去即可：  
*Jackson*  
``` java
/* JacksonRequest */
VolleyRequest.getInstance().newJacksonRequest("http://www.xxx.com/xxx", new TypeReference<List<MeteringHistoryEntity>>() {},
List<MeteringHistoryEntity>{
    @Override
    public void onResponse(List<MeteringHistoryEntity> list)
    {
        //get...
    }
}, new Response.ErrorListener()
{
    @Override
    public void onErrorResponse(VolleyError error)
    {
        DebugLog.e("JacksonRequest: " + error);
    }
});
}
```  
*Gson*
``` java
/* GsonRequest */
VolleyRequest.getInstance().newGsonRequest("http://www.xxx.com/xxx", new TypeToken<List<Map<String, Object>>>() {} .getType(),
new Response.Listener<List<Map<String, Object>>>()
{
    @Override
    public void onResponse(List<Map<String, Object>> list)
    {
       //get...
    }
}, new Response.ErrorListener()
{
    @Override
    public void onErrorResponse(VolleyError error)
    {
        DebugLog.e("GsonRequest: " + error);
    }
});
```  
# END
源码在我github上[AndroidUtils](https://github.com/zhengxiaopeng/AndroidUtils)里的[volley包](https://github.com/zhengxiaopeng/AndroidUtils/tree/master/library/src/main/java/com/roc/http/volley)里，可以下载整个项目源码来运行，或者直接看volley里的类即可。  
以上的Request返回打印结果：  
![](http://rocko-blog.qiniudn.com/Android-Volley自定义Request-JacksonRequest、GsonRequest_1.jpg?imageView2/2/w/400/h/400/q/100)  
![](http://rocko-blog.qiniudn.com/Android-Volley自定义Request-JacksonRequest、GsonRequest_2.jpg?imageView2/2/w/400/h/400/q/100)  



