***Retrofit和Okhttp关系，Retrofit如何通过okhttp完成请求的？***

Retrofit网络请求的本质是okhttp完成的，而Retrofit仅负责网络请求接口的封装。

App应用程序通过 Retrofit 请求网络，实际上是使用 Retrofit 接口层封装请求参数、Header、Url 等信息，之后由 OkHttp 完成后续的请求操作。

在服务端返回数据之后，OkHttp 将原始的结果交给 Retrofit，Retrofit根据用户的需求对结果进行解析。

