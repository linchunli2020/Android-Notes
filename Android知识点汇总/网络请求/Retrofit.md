***Retrofit和Okhttp关系，Retrofit如何通过okhttp完成请求的？***

Retrofit网络请求的本质是okhttp完成的，而Retrofit仅负责网络请求接口的封装。

App应用程序通过 Retrofit 请求网络，实际上是使用 Retrofit 接口层封装请求参数、Header、Url 等信息，之后由 OkHttp 完成后续的请求操作。

在服务端返回数据之后，OkHttp 将原始的结果交给 Retrofit，Retrofit根据用户的需求对结果进行解析。

***Retrofit的使用步骤***

    1、添加Retrofit库的依赖
    2、创建接收服务器返回的数据的类
    3、创建用于描述网络请求的接口
    4、创建Retrofit的实例
    5、创建网络请求接口实例并配置网络请求参数
    6、发送网络请求(异步/同步)
    
***（1）在gradle加入Retrofit依赖库***

    implementation 'com.squareup.retrofit2:retrofit:2.1.0'

    不要忘记添加网络权限
    <uses-permission android:name="android.permission.INTERNET"/>

***（2）创建接收服务器返回数据的实体类Bean.java***

    package com.qinkl.retrofitdemo;
    public class Bean {
        public String code;
        public String message;
    }
    
***（3）创建用于描述网络请求的接口***

RxJava将http请求抽象成Java接口，采用注解的方式描述网络请求的参数和配置网络请求的相关信息
用动态代理方式动态将该接口翻译成一个HTTP请求，最后在执行HTTP请求
接口的每个方法的参数都需要使用注解来标注，否则会报错

创建接口文件MyApiService.java

    package com.qinkl.retrofitdemo;
    import retrofit2.Call;
    import retrofit2.http.GET;

    public interface MyApiService {
        @GET("openapi.do?keyfrom=Yanzhikai&key=2032414398&type=data&doctype=json&version=1.1&q=car")
        Call<Bean> getCall();
    }
    
***（4）创建Retrofit实例***

    Retrofit retrofit = new Retrofit.Builder()
            .baseUrl("http://fy.iciba.com/")
            .addConverterFactory(GsonConverterFactory.create(new Gson()))
            .build();
    由于这里运用了GsonConverterFactory，所以要在gradle引入依赖包:

    compile 'com.squareup.retrofit2:converter-gson:2.0.2'
    
***（5）创建网络请求接口实例和配置网络请求参数相关信息***

    MyApiService service = retrofit.create(MyApiService.class);
    
***（6）发送网络请求***

    Call<Bean> call = service.getCall();

    call.enqueue(new Callback<Bean>() {
        @Override
        public void onResponse(Call<Bean> call, Response<Bean> response) {
            response.body().show();
        }

        @Override
        public void onFailure(Call<Bean> call, Throwable t) {
            Log.i("MainActivity","请求失败");
        }
    });
    

链接：https://www.jianshu.com/p/da3eeb2e2c9b

