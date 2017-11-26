##优点
* 物理量小

 > Volley.jar(120k)，加上自己的封装最多140k；OkHttp需要 okio.jar (80k), okhttp.jar(330k)这2个jar包，总大小差不多400k,加上自己的封装，差不多得410k。

* 非常适合进行数据量不大，但通信频繁的网络操作
* 扩展性强
* 可直接在主线程调用服务端并处理返回结果
 
 > ExecutorDelivery 封装了Handler的处理 在Runnable里面判断了Request状态

* 网络请求线程NetworkDispatcher默认开启了4个

* 通过使用标准的HTTP缓存机制保持磁盘和内存响应的一致
 
```
private static final int DEFAULT_POOL_SIZE = 4096; 
public BasicNetwork(HttpStack httpStack) {
// If a pool isn't passed in, then build a small default pool that will give us a lot of
// benefit and not use too much memory.
this(httpStack, new ByteArrayPool(DEFAULT_POOL_SIZE));
}
```

##缺点
* 默认 Android2.3 及以上基于 HttpURLConnection，2.3 以下基于 HttpClient 实现。
6.0不支持httpclient了，如果想支持得添加org.apache.http.legacy.jar
* 对大文件下载 Volley的表现非常糟糕。
* 只支持http请求。
* 图片加载性能一般。

##使用方法
* 使用Https 需自己添加代码
* cookie的未封装
* 对回调数据作了泛型处理
```
RequestQueue mQueue = Volley.newRequestQueue(context);  
StringRequest stringRequest = new StringRequest("http://www.baidu.com",  
                        new Response.Listener<String>() {  
                            @Override  
                            public void onResponse(String response) {  
                                Log.d("TAG", response);  
                            }  
                        }, new Response.ErrorListener() {  
                            @Override  
                            public void onErrorResponse(VolleyError error) {  
                                Log.e("TAG", error.getMessage(), error);  
                            }  
                        });  
mQueue.add(stringRequest);                          
```
## 源码解析
### Volley.java
```
//初始化RequestQueue
 public static RequestQueue newRequestQueue(Context context, BaseHttpStack stack) {
        BasicNetwork network;
        if (stack == null) {
            if (Build.VERSION.SDK_INT >= 9) {
                //大于SDK 9 使用HttpUrlConnection
                network = new BasicNetwork(new HurlStack());
            } else {
                // Prior to Gingerbread, HttpUrlConnection was unreliable.
                // See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
                // At some point in the future we'll move our minSdkVersion past Froyo and can
                // delete this fallback (along with all Apache HTTP code).
                String userAgent = "volley/0";
                try {
                    String packageName = context.getPackageName();
                    PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
                    userAgent = packageName + "/" + info.versionCode;
                } catch (NameNotFoundException e) {
                }
                //小于SDK 9 使用HttpClient
                network = new BasicNetwork(
                        new HttpClientStack(AndroidHttpClient.newInstance(userAgent)));
            }
        } else {
            network = new BasicNetwork(stack);
        }
        return newRequestQueue(context, network);
    }
private static RequestQueue newRequestQueue(Context context, Network network) {
        File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);
        RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
        queue.start();
        return queue;
    }
```
### RequestQueue

```
//线程池大小为4
private static final int DEFAULT_NETWORK_THREAD_POOL_SIZE = 4;
private final NetworkDispatcher[] mDispatchers;
public RequestQueue(Cache cache, Network network) {
        this(cache, network, DEFAULT_NETWORK_THREAD_POOL_SIZE);
    }
public RequestQueue(Cache cache, Network network, int threadPoolSize,
            ResponseDelivery delivery) {
        mCache = cache;
        mNetwork = network;
        mDispatchers = new NetworkDispatcher[threadPoolSize];
        mDelivery = delivery;
    }    
    
public void start() {
        stop();  // Make sure any currently running dispatchers are stopped.
        // Create the cache dispatcher and start it.
        mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
        mCacheDispatcher.start();

        // Create network dispatchers (and corresponding threads) up to the pool size.
        for (int i = 0; i < mDispatchers.length; i++) {
            NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
                    mCache, mDelivery);
            mDispatchers[i] = networkDispatcher;
            networkDispatcher.start();
        }
    }    
public <T> Request<T> add(Request<T> request) {
        // Tag the request as belonging to this queue and add it to the set of current requests.
        request.setRequestQueue(this);
        synchronized (mCurrentRequests) {
            mCurrentRequests.add(request);
        }

        // Process requests in the order they are added.
        request.setSequence(getSequenceNumber());
        request.addMarker("add-to-queue");

        // If the request is uncacheable, skip the cache queue and go straight to the network.
        if (!request.shouldCache()) {
            mNetworkQueue.add(request);
            return request;
        }
        mCacheQueue.add(request);
        return request;
     }

```

### BasicNetwork.java
```
public NetworkResponse performRequest(Request<?> request) throws VolleyError {
        long requestStart = SystemClock.elapsedRealtime();
        while (true) {
            HttpResponse httpResponse = null;
            byte[] responseContents = null;
            List<Header> responseHeaders = Collections.emptyList();
            try {
                // Gather headers.
                Map<String, String> additionalRequestHeaders =
                        getCacheHeaders(request.getCacheEntry());
                //executeRequest  HurlStack
                httpResponse = mBaseHttpStack.executeRequest(request, additionalRequestHeaders);
                int statusCode = httpResponse.getStatusCode();

                responseHeaders = httpResponse.getHeaders();
                // Handle cache validation.
                if (statusCode == HttpURLConnection.HTTP_NOT_MODIFIED) {
                    Entry entry = request.getCacheEntry();
                    if (entry == null) {
                        return new NetworkResponse(HttpURLConnection.HTTP_NOT_MODIFIED, null, true,
                                SystemClock.elapsedRealtime() - requestStart, responseHeaders);
                    }
                    // Combine cached and response headers so the response will be complete.
                    List<Header> combinedHeaders = combineHeaders(responseHeaders, entry);
                    return new NetworkResponse(HttpURLConnection.HTTP_NOT_MODIFIED, entry.data,
                            true, SystemClock.elapsedRealtime() - requestStart, combinedHeaders);
                }

                // Some responses such as 204s do not have content.  We must check.
                InputStream inputStream = httpResponse.getContent();
                if (inputStream != null) {
                  responseContents =
                          inputStreamToBytes(inputStream, httpResponse.getContentLength());
                } else {
                  // Add 0 byte response as a way of honestly representing a
                  // no-content request.
                  responseContents = new byte[0];
                }

                // if the request is slow, log it.
                long requestLifetime = SystemClock.elapsedRealtime() - requestStart;
                logSlowRequests(requestLifetime, request, responseContents, statusCode);

                if (statusCode < 200 || statusCode > 299) {
                    throw new IOException();
                }
                return new NetworkResponse(statusCode, responseContents, false,
                        SystemClock.elapsedRealtime() - requestStart, responseHeaders);
            } catch (SocketTimeoutException e) {
                attemptRetryOnException("socket", request, new TimeoutError());
            } catch (MalformedURLException e) {
                throw new RuntimeException("Bad URL " + request.getUrl(), e);
            } catch (IOException e) {
                int statusCode;
                if (httpResponse != null) {
                    statusCode = httpResponse.getStatusCode();
                } else {
                    throw new NoConnectionError(e);
                }
                VolleyLog.e("Unexpected response code %d for %s", statusCode, request.getUrl());
                NetworkResponse networkResponse;
                if (responseContents != null) {
                    networkResponse = new NetworkResponse(statusCode, responseContents, false,
                            SystemClock.elapsedRealtime() - requestStart, responseHeaders);
                    if (statusCode == HttpURLConnection.HTTP_UNAUTHORIZED ||
                            statusCode == HttpURLConnection.HTTP_FORBIDDEN) {
                        attemptRetryOnException("auth",
                                request, new AuthFailureError(networkResponse));
                    } else if (statusCode >= 400 && statusCode <= 499) {
                        // Don't retry other client errors.
                        throw new ClientError(networkResponse);
                    } else if (statusCode >= 500 && statusCode <= 599) {
                        if (request.shouldRetryServerErrors()) {
                            attemptRetryOnException("server",
                                    request, new ServerError(networkResponse));
                        } else {
                            throw new ServerError(networkResponse);
                        }
                    } else {
                        // 3xx? No reason to retry.
                        throw new ServerError(networkResponse);
                    }
                } else {
                    attemptRetryOnException("network", request, new NetworkError());
                }
            }
        }
    }

```

### HurlStack.java
```
 public HttpResponse executeRequest(Request<?> request, Map<String, String> additionalHeaders)
            throws IOException, AuthFailureError {
        String url = request.getUrl();
        HashMap<String, String> map = new HashMap<>();
        map.putAll(request.getHeaders());
        map.putAll(additionalHeaders);
        if (mUrlRewriter != null) {
            String rewritten = mUrlRewriter.rewriteUrl(url);
            if (rewritten == null) {
                throw new IOException("URL blocked by rewriter: " + url);
            }
            url = rewritten;
        }
        URL parsedUrl = new URL(url);
        HttpURLConnection connection = openConnection(parsedUrl, request);
        for (String headerName : map.keySet()) {
            connection.addRequestProperty(headerName, map.get(headerName));
        }
        setConnectionParametersForRequest(connection, request);
        // Initialize HttpResponse with data from the HttpURLConnection.
        int responseCode = connection.getResponseCode();
        if (responseCode == -1) {
            // -1 is returned by getResponseCode() if the response code could not be retrieved.
            // Signal to the caller that something was wrong with the connection.
            throw new IOException("Could not retrieve response code from HttpUrlConnection.");
        }

        if (!hasResponseBody(request.getMethod(), responseCode)) {
            return new HttpResponse(responseCode, convertHeaders(connection.getHeaderFields()));
        }

        return new HttpResponse(responseCode, convertHeaders(connection.getHeaderFields()),
                connection.getContentLength(), inputStreamFromConnection(connection));
    }
```

##缓存
###DiskBasedCache.java
```
    //默认缓存大小为5M
    /** Default maximum disk usage in bytes. */
    private static final int DEFAULT_DISK_USAGE_BYTES = 5 * 1024 * 1024;
    //写入文件
```