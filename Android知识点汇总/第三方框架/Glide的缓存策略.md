***1.Glide的缓存机制简介：***

**Glide采取的多级缓存机制，将缓存分为2大部分：内存缓存，硬盘缓存；其中内存缓存又分为2种：弱引用和Lrucache。**

    内存缓存的主要作用：防止应用重复将图片数据读取到内存中
    硬盘缓存的主要作用：防止应用重复从网络或其它地方下载和读取数据
    
由此可见，以APP应用范围来看，内存缓存主要针对内在处理，硬盘缓存主要针对对外管理，也由二者结合才构成Glide的主要缓存机制基础。

默认情况下，内存缓存和硬件缓存，Glide都是开启的。

当然官方也提供了二者的开关设置：

    skipMemoryCache()方法并传入true，就表示禁用掉Glide的内存缓存功能；
    调用diskCacheStrategy()方法并传入DiskCacheStrategy.NONE，就可以禁用掉Glide的硬盘缓存功能了。
    （这里并不是boolean值类型，因为Glide提供了四种枚举类型，在1.3硬盘缓存中会进行说明）。
    
***1.1 Key的生成***

缓存机制中最主要的问题之一就是避免重复加载，那要作为避免重复就需要有依据，这个依据就是 缓存Key，
它是每个图的多项信息经过Glide内部算法生成的（其中一个重要参数信息：model对应的就是资来源，如最常见的网络资源URL，这也造成动态URL的时候会发生重复加载的问题，
这我们会在第4部分进行说明）。

在4.11版本中获取缓存key基本原理仍旧是通过多种信息（请求资源、signature、width、height等），通过hashCode的计算得到一个单独的标志。 
在源码中我们可以追溯，知道RequestBuilder中对应的model，可以知道它对应来源于load，从此可知，model其实就是我们的请求资源（url、file等）。源码如下：

<img width="659" alt="image" src="https://user-images.githubusercontent.com/67937122/162374985-0722a66f-a646-4f47-9a66-29dec0846192.png">

然后，这个model会结合signature、width、height等多种参数一起传入到EngineKeyFactory的buildKey()方法当中，从而构建出了一个EngineKey对象，
这个EngineKey也就是Glide中的缓存Key了。

因此，如果你图片的width或者height发生改变，也会生成一个完全不同的缓存Key。

（这里补充一下：4.4以前是Bitmap复用必须长宽相等才可以复用，而4.4及以后是Size>=所需就可以复用，只不过需要调用reconfigure来调整尺寸）

***1.2 内存缓存***

当Glide加载完某个图片后，就会将它放入内存缓存中，在它被内存缓存回收之前，再调用就可以直接从内存缓存里直接获取。

当然，这是最基本的运用场景，其内存缓存机制里所依据的算法是 LRUCache算法。

而且，为了达到更好的效果，还采取了弱引用机制，结合LRUCache，二者奠定了内存缓存的总体基础。

Glide将内存缓存中划分成两个区域：

    LruResourceCache（某些人称为 图片池，就是Glide实现内存缓存所使用的LruCache对象了）+
    activeResources（某些人称为对象池，即正在使用的图片管理处）。
    
由此可知，前者使用的是LRU算法进行管理的缓存工具，而activeResources就是使用了弱引用机制（采取了HashMap进行弱引用进行存储）。

**默认情况下，Glide会在开始一个新的图片请求之前检查以下多级的缓存：（Glide请求新图片的过程）**

Glide检查开启内存缓存后，如果开启了，先去activeResources中进行搜索，如果找到了直接使用，并将引用对象索引+1（active.acquire();）。
找不到再去LruResourceCache中进行搜索，如果在LruResourceCache中找到了，则将其从LruResourceCache中移除，并放入activeResources中。
源码如下：

<img width="665" alt="image" src="https://user-images.githubusercontent.com/67937122/162375926-cf471dd8-ee8d-4f27-b548-28f9f66990e2.png">

<img width="666" alt="image" src="https://user-images.githubusercontent.com/67937122/162375950-57f5d625-042b-4398-9abc-cc54c85829ca.png">

如果二者均找不到符合资源，再开启子线程进行加载图片资源。由此可知，在Glide的内存缓存机制中，LruResourceCache的优先级在activeResources之前。


    另外，在Glide内存缓存中，通过EngineResource作为图片的管理对象，里面有一个参数变量acquired用来记录图片被引用的次数，
    调用acquire()方法会让变量加1，调用release()方法会让变量减1。
    当acquired变量大于0的时候，说明图片正在使用中，也就应该放到activeResources弱引用缓存当中。
    而经过release()之后，如果acquired变量等于0了，说明图片已经不再被使用了，此时会进行内部回收。

    这里的内部回收过程是这样的：首先会将缓存图片从activeResources中移除，然后再将它put到LruResourceCache当中。
    这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用LruCache来进行缓存的功能。源码如下：
    
<img width="666" alt="image" src="https://user-images.githubusercontent.com/67937122/162376895-8f059d71-8579-4bf9-9d57-d60053952f44.png">


***1.3 硬盘缓存***

**硬盘缓存的资源管理机制：**

其实Glide在这里也是采用LRU算法进行管理。上头说到，Glide在内存缓存中没有找到对应的图片资源后，便会开启新的子线程进行图片加载，

此时会执行EngineRunnable的run()方法，run()方法中又会调用一个decode()方法：

<img width="645" alt="image" src="https://user-images.githubusercontent.com/67937122/162377860-626df52b-fa49-48e3-88cc-4105ffb7392f.png">

可知，Glide会先在硬盘缓存中进行搜索，当硬盘缓存存在后直接读取缓存的图片（decodeFromCache方法），当硬盘缓存中不存在符合资源时候，
再调用decodeFromSource()来读取原始图片。而在硬盘缓存读取中（decodeFromCache方法）先去调用DecodeJob的decodeResultFromCache()方法来获取缓存，
如果获取不到，会再调用decodeSourceFromCache()方法获取缓存，这两个方法的区别其实就是DiskCacheStrategy.RESULT和DiskCacheStrategy.SOURCE这两个参数的区别，
而这两个指的就是硬盘缓存是采取“转换后的图片缓存”、“原始图片缓存”的选择。

那什么时候写入硬盘缓存？简单说一下，默认下是在网络请求图片，进行转化后，写入硬盘缓存的。

***2.Glide 的请求流程***

这里，我们从最直接的使用角度进行解释：

**1. Glide.with(context)创建RequestManager**

RequestManager负责管理当前context的所有Request

Context可以传Fragment、Activity或者其他Context，当传Fragment、Activity时，当前页面对应的Activity的生命周期可以被RequestManager监控到，
从而可以控制Request的pause、resume、clear。**这其中采用的监控方法就是在当前activity中添加一个没有view的fragment，
这样在activity发生onStart onStop onDestroy的时候，会触发此fragment的onStart onStop onDestroy。**

RequestManager用来跟踪众多当前页面的Request的是RequestTracker类，**用弱引用来保存运行中的Request，用强引用来保存暂停需要恢复的Request。**

**2. Glide.with(context).load(url)创建需要的Request**

通常是DrawableTypeRequest，后面可以添加transform、fitCenter、animate、placeholder、error、override、skipMemoryCache、signature等等

如果需要进行Resource的转化比如转化为Byte数组等需要，可以加asBitmap来更改为BitmapTypeRequest

**Request是Glide加载图片的执行单位**

**3. Glide.with(context).load(url).into(imageview)**

在Request的into方法中会调用Request的begin方法开始执行

在正式生成EngineJob放入Engine中**执行之前，如果并没有事先调用override(width, height)来指定所需要宽高，Glide则会尝试去获取imageview的宽和高，
如果当前imageview并没有初始化完毕取不到高宽，Glide会通过view的ViewTreeObserver来等View初始化完毕之后再获取宽高再进行下一步。**

 ***3.常见的压缩方式***
 
 Android中图片是以bitmap形式存在的，那么bitmap所占内存，直接影响到了应用所占内存大小，首先要知道计算方式：

**bitmap所占内存大小 = 图片长度 * 图片宽度 * 一个像素点占用的字节数（单位尺寸的像素密度一般是手机分辨率所决定的）**

因此我们常见的压缩方式就是两种方式：

    1.将图片的w、h进行压缩；
    2.降低像素点占用的字节数。而我们最常见的是尺寸压缩，质量压缩和格式压缩，这里我们简单解释一下它们。

**3.1 尺寸压缩&采样率压缩**

    说实话，个人感觉这两个压缩概念上虽然有差异，但是实际使用中原理是相通的，或者说二者几乎都是一起使用的。

    尺寸压缩就是将图片的大小（w、h进行缩小），从而保持单位尺寸上的像素密度不变，由此像素点的减少，使得图片大小得到减少；

    采样率压缩通过设置采样率,单位尺寸上减少像素密度（比如读取图片时候，并不读取所有的像素点，只读取部分像素点），造成单位尺寸上的像素密度降低, 
    达到对内存中的Bitmap进行压缩，或者说就是按照一定的倍数对图片减少单位尺寸的像素值。其原理为：通过减少单位尺寸的像素值，真正意义上的降低像素值。
    但是一般为了保持图片不失真，会结合尺寸的压缩。

    常见的使用场景：缓存缩略图 (头像的处理)。

    通常通过BitmapFactory中的decodeFile方法对图片进行尺寸压缩，其中一个参数opts 就是所谓的采样率，它里边有很多属性可以设置，我们通过设置属性来达到根据自己的需要，压缩出指定的图片。

    为了避免OOM异常，最好在解析每张图片的时候都先检查一下图片的大小，除非你非常信任图片的来源，保证这些图片都不会超出你程序的可用内存。

    现在图片的大小已经知道了，我们就可以决定是把整张图片加载到内存中还是加载一个压缩版的图片到内存中。以下几个因素是我们需要考虑的：

    1. 预估一下加载整张图片所需占用的内存。

    2. 为了加载这一张图片你所愿意提供多少内存。

    3. 用于展示这张图片的控件的实际大小。

    4. 当前设备的屏幕尺寸和分辨率。

    比如，你的ImageView只有128*96像素的大小，只是为了显示一张缩略图，这时候把一张1024*768像素的图片完全加载到内存中显然是不值得的。

    那我们怎样才能对图片进行压缩呢？通过设置BitmapFactory.Options中inSampleSize的值就可以实现。比如我们有一张2048*1536像素的图片，将inSampleSize的值设置为4，就可以把这张图片压缩成512*384像素。原本加载这张图片需要占用13M的内存，压缩后就只需要占用0.75M了(假设图片是ARGB_8888类型，即每个像素点占用4个字节)。下面的方法可以根据传入的宽和高，计算出合适的inSampleSize值

**3.2 质量压缩**

    顾名思义就是降低图片质量（大小）。原理 ：通过算法扣掉（同化）了 图片中的一些某个点附近相近的像素，达到降低质量减少文件大小的目的。

**注意 ： 通过上述我们知道，它并没有改变w、h,也没有单个像素占据的字节数。所以只能实现对 file 的影响，对加载这个图片出来的bitmap 内存是无法节省的，
因为质量压缩不会减少图片的像素，它是在保持像素的前提下改变图片的位深及透明度等，来达到压缩图片的目的。**

    使用场景 ：将图片保存到本地 ，或者将图片上传 到服务器 ，根据实际需求来 。

    质量压缩主要借助Bitmap中的compress方法实现：

    public boolean compress (Bitmap.CompressFormat format, int quality, OutputStream stream)

    这个方法用来将特定格式的压缩图片写入输出流（OutputStream）中，当然例如输出流与文件联系在一起，压缩后的图片也就是一个文件。如果压缩成功则返回true，其中有三个参数：

    format是压缩后的图片的格式，可取值：Bitmap.CompressFormat .JPEG、~.PNG、~.WEBP。

    quality的取值范围为[0,100]，值越小，经过压缩后图片失真越严重，当然图片文件也会越小。（PNG格式的图片会忽略这个值的设定）

    stream指定压缩的图片输出的地方，比如某文件。

    上述方法还有一个值得注意的地方是：当用BitmapFactory decode文件时可能返回一个跟原图片不同位深的图片，或者丢失了每个像素的透明值（alpha），比如说，JPEG格式的图片仅仅支持不透明的像素。

**3.3 RGB_565压缩**

    这里先说明一下位图的一些知识点：

    ARGB_8888：表示32位ARGB位图，即A=8,R=8,G=8,B=8,一个像素点占8+8+8+8=32位，4个字节；

    RGB_565：表示16位RGB位图,即R=5,G=6,B=5,它没有透明度,一个像素点占5+6+5=16位，2个字节；

    其实就是牺牲图片的透明度（此时一个像素点占用的字节数得到降低），这样压缩出来的图片比ARGB_8888能达到（并不一定是）一半的内存开销。
    个人感觉（如果有误）通过将图片进行格式转化——格式压缩，通过的主要也是这种方法，尤其JPEG格式的图片仅仅支持不透明的像素，png格式的图片几乎不改变图片大小。
    另外，常见的jpeg和png都有损压缩，若非必要，尽量别来回压缩。图像压缩很费时间和内存的，特别是时间。

    使用场景 ：无需在意图片的透明度时候。

    其实，网上很多都有具体的代码示例，但是多数都是调用已有的API进行说明，目前看到的解释都比较乱一点，活着说本人现在理解的也有点乱。暂时能够归置出这一点点压缩的知识，本人将十万分地希望和感谢有小伙伴能够指正或者指点！！！

***4.动态URL与Glide缓存机制冲突问题***

这个问题我直接把参看博客截图放上来把：
    
<img width="665" alt="image" src="https://user-images.githubusercontent.com/67937122/162382685-e68b6f16-fae2-4cc6-8145-e6668a398f32.png">

Glide对于一个图片会通过图片资源的多项信息，通过本身算法得到一个key，这个key就是Glide作为对图片资源的唯一性标识，其中一个参数就是图片的URL。
    
由此可见，当我们的URL发生变化的时候，对应的key也会发生变化。这样子就会造成上图所述的问题，获取key的因素在上头我们已经说了，就不赘述了。
    

也因此在对应的model（就是我们的请求资源），我们需要在GlideUrl类中修改对应的getCacheKey方法去修改key生成规则。
    

接下来我们就要看一下GlideUrl的getCacheKey()方法的源码了，如下所示：

        public class GlideUrl {

            private final URL url;
            private final String stringUrl;
            ...

            public GlideUrl(URL url) {
                this(url, Headers.DEFAULT);
            }

            public GlideUrl(String url) {
                this(url, Headers.DEFAULT);
            }

            public GlideUrl(URL url, Headers headers) {
                ...
                this.url = url;
                stringUrl = null;
            }

            public GlideUrl(String url, Headers headers) {
                ...
                this.stringUrl = url;
                this.url = null;
            }

            public String getCacheKey() {
                return stringUrl != null ? stringUrl : url.toString();
            }

            ...
        }
        
这里我将代码稍微进行了一点简化，这样看上去更加简单明了。
GlideUrl类的构造函数接收两种类型的参数，一种是url字符串，一种是URL对象。
然后getCacheKey()方法中的判断逻辑非常简单，如果传入的是url字符串，那么就直接返回这个字符串本身，如果传入的是URL对象，那么就返回这个对象toString()后的结果。

其实看到这里，我相信大家已经猜到解决方案了，因为getCacheKey()方法中的逻辑太直白了，直接就是将图片的url地址进行返回来作为缓存Key的。
那么其实我们只需要重写这个getCacheKey()方法，加入一些自己的逻辑判断，就能轻松解决掉刚才的问题了。

创建一个MyGlideUrl继承自GlideUrl，代码如下所示：

        public class MyGlideUrl extends GlideUrl {

            private String mUrl;

            public MyGlideUrl(String url) {
                super(url);
                mUrl = url;
            }

            @Override
            public String getCacheKey() {
                return mUrl.replace(findTokenParam(), "");
            }

            private String findTokenParam() {
                String tokenParam = "";
                int tokenKeyIndex = mUrl.indexOf("?token=") >= 0 ? mUrl.indexOf("?token=") : mUrl.indexOf("&token=");
                if (tokenKeyIndex != -1) {
                    int nextAndIndex = mUrl.indexOf("&", tokenKeyIndex + 1);
                    if (nextAndIndex != -1) {
                        tokenParam = mUrl.substring(tokenKeyIndex + 1, nextAndIndex + 1);
                    } else {
                        tokenParam = mUrl.substring(tokenKeyIndex);
                    }
                }
                return tokenParam;
            }

        }

可以看到，这里我们重写了getCacheKey()方法，在里面加入了一段逻辑用于将图片url地址中token参数的这一部分移除掉。
这样getCacheKey()方法得到的就是一个没有token参数的url地址，从而不管token怎么变化，最终Glide的缓存Key都是固定不变的了。

当然，定义好了MyGlideUrl，我们还得使用它才行，将加载图片的代码改成如下方式即可：

        Glide.with(this)
             .load(new MyGlideUrl(url))
             .into(imageView);

也就是说，我们需要在load()方法中传入这个自定义的MyGlideUrl对象，而不能再像之前那样直接传入url字符串了。
不然的话Glide在内部还是会使用原始的GlideUrl类，而不是我们自定义的MyGlideUrl类。


“解决Glide生成缓存Key问题”具体解决可参考链接：https://blog.csdn.net/huangxiaoguo1/article/details/78554107

链接：https://www.jianshu.com/p/78ad4ce1694e

