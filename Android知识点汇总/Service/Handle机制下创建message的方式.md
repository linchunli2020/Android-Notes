我们在使用Handler通信的时候，通常还会创建一个Message对象用于对消息的封装。那创建一个Message对象有哪些方式呢？

我们常用的创建Message对象的方式有如下三种:

<img width="546" alt="image" src="https://user-images.githubusercontent.com/67937122/162691632-bc96f349-8869-415f-bf69-af4298e6386d.png">

**其中方式1，是我们最常用的方式，通过Message的无参构造函数创建一个Message对象：**

<img width="271" alt="image" src="https://user-images.githubusercontent.com/67937122/162691693-226c931b-1627-460d-9f16-580eca9c495b.png">

这个没什么好说的，我们每new一次，就会在内存中创建一个Message对象。

**方式2，Message.obtain()及其系列的重载方法，我们先看看他的无参方法的源码实现：**

<img width="850" alt="image" src="https://user-images.githubusercontent.com/67937122/162691796-db90510d-b088-4a44-b1be-b5e75c798b52.png">

sPoolSync参数是一个Object对象，用来同步保证线程安全。sPool是一个Message 对象，sPoolSize是一个整体上控制Message对象数量的参数，防止通过该方法创建的Message对象过多。
另外，与之相关联的方法还有以下几个方法：

      public void recycle() {
            if (isInUse()) {
                if (gCheckRecycle) {
                    throw new IllegalStateException("This message cannot be recycled because it "
                            + "is still in use.");
                }
                return;
            }
            recycleUnchecked();
        }

        void recycleUnchecked() {
            // 当我们使用完Message对象的时候，系统就回收Message对象，但这里的回收跟JVM虚拟机的回收不一样，
            //他并不是直接将Message对象置为空，而是将Message对象里面的各参数置为初始状态。
            flags = FLAG_IN_USE;
            what = 0;
            arg1 = 0;
            arg2 = 0;
            obj = null;
            replyTo = null;
            sendingUid = UID_NONE;
            workSourceUid = UID_NONE;
            when = 0;
            target = null;
            callback = null;
            data = null;
            synchronized (sPoolSync) {
                if (sPoolSize < MAX_POOL_SIZE) {
                    next = sPool;
                    sPool = this;
                    //完成回收后，再将Message对象添加到池子里面去，恢复数量，方便下次获取。
                    sPoolSize++;
                }
            }
        }
        
        
这两个方法的作用就是当我们使用完通过方式2创建的Message对象后，系统会自动回收Message对象，但这里的回收并不是直接将Message对象清空置为null，
而是清空Message对象里面的所有信息，然后再将其添加到系统的sPoolSize中去，方便下次直接使用。

简单的说，使用方式2创建Message对象的好处是Message对象可以重复使用,可以免除一直new Message对象造成无谓的内存压力(不断新建销毁对象),实现 类似于线程池的功能。

**方式2也是Google推荐的方式，Handler的send系列方法和post系列方法内部的message对象都是使用这个方式获取的。**

**方式3、new Handler().obtainMessage()及其系列的重载方法**，我们先看看他的无参方法的源码实现：

      public final Message obtainMessage(){
            return Message.obtain(this);
      }
      
      
Message.obtain(this)方法具体 如下：

      public static Message obtain(Handler h) {
            Message m = obtain();
            m.target = h;//在这里，实现了Message对象与Handler对象的绑定
            return m;
      }

我们可以看到，new Handler().obtainMessage()方法内部最终还是调用了obtain()方法，又回到了方式2，这跟方式2创建Message对象是一样的，本质上没有区别。

因此我们在创建Message对象时，尽可能的使用方式2，以减少内存开销。


链接：https://blog.csdn.net/haoyuegongzi/article/details/102984557

