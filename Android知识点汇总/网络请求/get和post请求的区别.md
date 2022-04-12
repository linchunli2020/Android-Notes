    (1) get 是从服务器上获取数据，post 是向服务器传送数据。 get 请求返回 request - URI 所指出的任意信息。

        Post 请求用来发送电子邮件、新闻或发送能由交互用户填写的表格。这是唯一需要在请求中发送body的请求。

        使用Post请求时需要在报文首部 Content - Length 字段中指出body的长度。
        

    (2) get 是把参数数据队列加到提交表单的ACTION属性所指的URL中，值和表单内各个字段一一对应，在URL中可以看到。

        post是通过HTTP post机制，将表单内各个字段与其内容放置在HTML HEADER内一起传送到ACTION属性所指的URL地址，用户看不到这个过程。
        

    (3) 对于 get 方式，服务器端用Request.QueryString获取变量的值，对于 post 方式，服务器端用Request.Form获取提交的数据。
    

    (4) get 传送的数据量较小，不能大于2KB。post 传送的数据量较大，一般被默认为不受限制。但理论上，IIS4中最大量为80KB，IIS5中为100KB。 

        用IIS过滤器的只接受get参数，所以一般大型搜索引擎都是用get方式。
        
    
    (5) get 安全性非常低，post 安全性相对较高。如果这些数据是中文数据而且是非敏感数据，那么使用 get；

        如果用户输入的数据不是中文字符而且包含敏感数据，那么还是使用 post 为好。


链接：https://blog.csdn.net/u010236550/article/details/28271783
