***（一）概念***

在OkHttp内部是使用拦截器来完成请求和响应，利用的是责任链设计模式，能够用来转换，重试，重写请求的机制。如今主流的网络框架非Retrofit莫属，它的内部请求也是基于OkHttp的。

在每个拦截器中，一个关键部分就是使用chain.proceed(request)发起请求。这个简单的方法就是全部Http工做发生的地方，生成和请求对应的响应。

多个拦截器能够连接使用。假设一个压缩拦截器和一个校验和拦截器：须要决定数据是否先被压缩，而后校验或者先校而后再压缩。OkHttp使用列表来跟踪拦截器，并按顺序调用拦截器。

<img width="814" alt="image" src="https://user-images.githubusercontent.com/67937122/162715353-846679ec-0e2f-46f9-a6ff-eeecbe71f5a1.png">

如上图所示，就是OkHttp中数据流的传输方向，里面包含了两种拦截器：

    Application Interceptors应用程序拦截器
    Network Interceptors网络拦截器
    
    
***（二）分类***

**1.应用拦截器**

经过下面两种方式注册的为应用拦截器：

    //方式一：在OkHttpClient.Builder中添加
    new OkHttpClient.Builder().addInterceptor(interceptor)

    //方式二：在okHttpClient中直接添加
    okHttpClient.interceptors().add(interceptor)

主要用于查看请求信息及返回信息，如连接地址、头信息、参数信息等，以下面的log-拦截器定义：json

      class LoggingInterceptor implements Interceptor {

        @Override public Response intercept(Interceptor.Chain chain) throws IOException {

              Request request = chain.request();

              long t1 = System.nanoTime();

              logger.info(String.format("Sending request %s on %s%n%s", request.url(),

              chain.connection(), request.headers()));

              Response response = chain.proceed(request);

              long t2 = System.nanoTime();

              logger.info(String.format("Received response for %s in %.1fms%n%s",

              response.request().url(), (t2 - t1) / 1e6d, response.headers()));

              return response;

        }
        
      }
      
      
**2.网络拦截器设计模式**

经过下面两种方式注册的为网络拦截器：缓存

    //方式一：在OkHttpClient.Builder中添加
    new OkHttpClient.Builder().addNetworkInterceptor(interceptor)

    //方式二：在okHttpClient中直接添加
    okHttpClient. networkInterceptors().add(interceptor)

能够添加、删除或替换请求头信息，还能够改变的请求携带的实体。网络

1)添加请求头，假设后台要求咱们请求API接口时，要在每个接口请求头上添加对应的Token。使用Retrofit的话可能条件反射出如下代码：

@FormUrlEncoded

@POST("/mobile/login.htm")

Call login(@Header("token") String token, @Field("mobile") String phoneNumber, @Field("smsCode") String smsCode);

这样写是能够，可是若是接口不少的话每个都须要传入token，重复不少遍，很容易致使代码的冗余。

下面使用拦截器就能够一劳永逸了，代码以下：

        public class TokenHeaderInterceptor implements Interceptor {

            @Override

            public Response intercept(Chain chain) throws IOException {

                // get token

                String token = AppService.getToken();

                Request originalRequest = chain.request();

                // get new request, add request header

                Request updateRequest = originalRequest.newBuilder()

                .header("token", token)

                .build();

                return chain.proceed(updateRequest);
             }

        }
        
先拦截获得originalRequest，而后利用originalRequest生成新的updateRequest，再交给chain处理进行下一环。最后在OkhttpClient中使用：

    OkHttpClient client = new OkHttpClient.Builder()

    .addNetworkInterceptor(new TokenHeaderInterceptor())

    .build();


    Retrofit retrofit = new Retrofit.Builder().baseUrl(BuildConfig.BASE_URL)

    .client(client).addConverterFactory(GsonConverterFactory.create()).build();


2)修改请求体

假设有如下需求：在上面login接口基础上，后台要求咱们传过去的请求参数要按照必定规则通过加密的。

规则以下：

    请求参数名统一为content；
    content值：JSON 格式的字符串通过 AES 加密后的内容；

举个例子，根据上面login接口，现有 {"mobile":"157xxxxxxxx", "smsCode":"xxxxxx"} Json字符串，而后再将其加密。
最后以content = [加密后json字符串]方式发送给后台。

看完了上面的TokenHeaderInterceptor以后，这需求对于咱们来讲能够算是信手拈来：

        public class RequestEncryptInterceptor implements Interceptor {

              private static final String FORM_NAME = "content";

              private static final String CHARSET = "UTF-8";

              @Override

              public Response intercept(Chain chain) throws IOException {

                      Request request = chain.request();

                      //获取请求体

                      RequestBody body = request.body();

                      if (body instanceof FormBody) {

                      FormBody formBody = (FormBody) body;

                      Map formMap = new HashMap<>();

                      // 从 formBody 中拿到请求参数，放入 formMap 中

                      for (int i = 0; i < formBody.size(); i++) {

                      formMap.put(formBody.name(i), formBody.value(i));

                      }
                      // 将 formMap经过Gson.toJson() 转化为 json 而后 AES 加密

                      Gson gson = new Gson();

                      String jsonParams = gson.toJson(formMap);

                      String encryptParams = AESCryptUtils.encrypt(jsonParams.getBytes(CHARSET), AppConstant.getAESKey());

                      // 从新修改 body 的内容

                      body = new FormBody.Builder().add(FORM_NAME, encryptParams).build();

                      }

                      // 若请求体不为Null，从新构建post请求，并传入修改后的参数体

                      if (body != null) {

                      request = request.newBuilder().post(body).build();

                      }

                      return chain.proceed(request);

              }

        }
        
        
***(三) 选择***

每一个拦截器都有各自的优势

**Application interceptors应用程序拦截器**

    不需要担忧好比重定向和重试的中间响应。
    
    老是被调用一次，即便HTTP响应结果是从缓存中获取的。
    
    监控应用程序的原始意图。不关心例如OkHttp注入的头部字段If-None-Match。
    
    容许短路，不调用Chain.proceed()。
    
    容许重试并屡次调用Chain.proceed()。

**Network Interceptors网络拦截器**

    可以对中间的响应进行操做好比重定向和重试。

    当发生网络短路时，不调用缓存的响应结果。

    监控数据，就像数据再网络上传输同样。

    访问承载请求的链接Connection。
    
    
链接：https://blog.csdn.net/weixin_39722921/article/details/117527680
