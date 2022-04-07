***(1)Evenbus***

定义：一个发布 / 订阅的事件总线，是一个开源库；

作用：选择使用它来进行模块间通信、解耦；

使用方式：EventBus库中最重要的三个点，分别是**subscriber（订阅者）**，**事件（消息）**，**publisher（发布者）**。主要理解这三者的关系即可。

    subscriber ——> EventBus 的register方法，传入的object对象
    事件（Event）——> EventBus 的post方法，传入的类型。
    publisher（发布者）——> EventBus的post方法。
    
第一步：创建一个事件，说白了就是创建一个类，也就是用来传递的东西，消息，类似bean，比如第二步的EvenMessage类；

第二步：在需要订阅事件的模块中，注册eventbus，在需要接受事件的类中进行好register之后，需要在该类中创建一个方法来接收事件消息，如下。在不用的时候千万别忘了unregister。

<img width="507" alt="image" src="https://user-images.githubusercontent.com/67937122/162134596-16412455-ec87-44f5-b3f8-4ae9e52cc786.png">


第三步：在需要发送事件的地方，调用EventBus的post（Object event），postSticky（Object event）来通知订阅者，如下：

<img width="560" alt="image" src="https://user-images.githubusercontent.com/67937122/162134619-224a5331-11d2-483e-809c-e95f812c9469.png">

***(2)Rxjava***

定义：实现异步操作的开源库；

作用：逻辑上（不一定是代码量），更为简洁地实现异步操作，特别是需求变复杂的时候，这一优势更加明显；

使用方式：RxJava 有四个基本概念：**Observable(可观察者，即被观察者)**、**Observer(观察者）**、**subscribe(订阅)**、**事件**。
Observable和Observer通过subscribe()方法实现订阅关系，从而Observable可以在需要的时候发出事件来通知Observer。
与传统观察者模式不同，RxJava 的事件回调方法除了普通事件onNext()（相当于onClick()/onEvent()）之外，还定义了两个特殊的事件：onCompleted()和onError()。

**第一步：创建 Observer。Observer 即观察者，它决定事件触发的时候将有怎样的行为。**

    法一：通过Observer接口；
    法二：通过Observer的抽象类：Subscriber，Subscriber对Observer接口进行了一些扩展，但他们的基本使用方式是完全一样的：

<img width="456" alt="image" src="https://user-images.githubusercontent.com/67937122/162134889-13ba735d-23f6-45aa-86c3-f81b05068006.png">

<img width="504" alt="image" src="https://user-images.githubusercontent.com/67937122/162134913-30d51e5e-2959-4697-8844-83d40fffe676.png">

**第二步：创建 Observable。Observable 即被观察者，它决定什么时候触发事件以及触发怎样的事件。RxJava 使用 create() 方法来创建一个Observable ，并为它定义事件触发规则：**

<img width="560" alt="image" src="https://user-images.githubusercontent.com/67937122/162134988-f4226d92-7130-4f29-a303-7ab06dfac6f8.png">

**第三步：Subscribe (订阅)。创建了 Observable 和 Observer 之后，再用 subscribe() 方法将它们联结起来，整条链子就可以工作了。**

<img width="292" alt="image" src="https://user-images.githubusercontent.com/67937122/162135047-2148559a-ce0a-4083-9594-16352c66e0d8.png">

    注意：在 RxJava 的默认规则中，事件的发出和消费都是在同一个线程的。也就是说，如果只用上面的方法，实现出来的只是一个同步的观察者模式。
    观察者模式本身的目的就是『后台处理，前台回调』的异步机制，因此异步对于 RxJava 是至关重要的。而要实现异步，则需要用到 RxJava 的另一个概念： Scheduler 。

<img width="559" alt="image" src="https://user-images.githubusercontent.com/67937122/162135219-c262bba7-a7eb-4ef8-8c97-920540aefbb7.png">


***(3)RxBus***

定义：一种模式，但它不是一个库，由Rxjava封装而来；

作用：类似EventBus，如果你的项目已经加入RxJava和EventBus，不妨用RxBus代替EventBus，以减少库的依赖。

使用方式：

**第一步：新建RxBus类；**

Subject同时充当了Observer和Observable的角色，Subject是非线程安全的，
在并发情况下，不推荐使用通常的Subject对象，而是推荐使用SerializedSubject，ofType操作符只发射指定类型的数据，其内部就是filter+cast。

<img width="434" alt="image" src="https://user-images.githubusercontent.com/67937122/162135329-b10f9842-1fad-43ec-8c20-83339cb8e5b0.png">

<img width="415" alt="image" src="https://user-images.githubusercontent.com/67937122/162135356-49cea452-888f-4082-9b9e-00989f631e68.png">

**第二步：创建需要发送的事件类**

<img width="342" alt="image" src="https://user-images.githubusercontent.com/67937122/162135422-41e422a8-48f3-4d81-ab64-1a8fb930c20f.png">

**第三步：发送事件**

<img width="384" alt="image" src="https://user-images.githubusercontent.com/67937122/162135469-b78fdc3d-883a-41d8-8f29-03cb4bb556fb.png">

**第四步：接收事件。**

rxSbscription是Sbscriptio的对象，我们这里把RxBus.getInstance().toObserverable(StudentEvent.class)
赋值给rxSbscription以方便生命周期结束时取消订阅事件.
      
<img width="553" alt="image" src="https://user-images.githubusercontent.com/67937122/162135542-422d051b-7580-475e-b115-4eadfbbc28af.png">

**第五步：取消订阅**

<img width="306" alt="image" src="https://user-images.githubusercontent.com/67937122/162135608-8f8379ec-8c85-4aac-8fc6-aa6e5622024c.png">



***(4)RxBus和EventBus区别是什么？***
RxJava 主要做异步、网络的数据处理，强大之处就是对数据的处理了，而对于处理完后的数据处理是一样的都是观察者模式来通知，也可以把 RxJava 进一步封装出一个 EventBus（RxBus）库，
二者可以转换的。
EventBus比较适合仅仅当做组件间的通讯工具使用，主要用来传递消息。使用EventBus可以避免搞出一大推的interface，仅仅是为了实现组件间的通讯，而不得不去实现那一推的接口。

原文链接：https://blog.csdn.net/WLX10428/article/details/104403590
