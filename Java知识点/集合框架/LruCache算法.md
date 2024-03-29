***LruCache（Least Recently Used）算法的核心思想就是最近最少使用算法。***

他在算法的内部维护了一个LinkHashMap的链表，通过put数据的时候判断是否内存已经满了，如果满了，则将最近最少使用的数据给剔除掉，从而达到内存不会爆满的状态。
这么讲可能有些抽象，我从网上找了一张图来解释这个算法。

<img width="759" alt="image" src="https://user-images.githubusercontent.com/67937122/161374899-747c1360-0f63-4067-aa39-3aadde5e4be7.png">

<img width="753" alt="image" src="https://user-images.githubusercontent.com/67937122/161374904-b5b3997c-8bad-4455-821f-84ac08fdc628.png">


通过上面这张图，我们可以看到，LruCache算法内部其实是一个队列的形式在存储数据，先进来的数据放在队列的尾部，后进来的数据放在队列头部，如果要使用队列中的数据，
那么使得之后将其又放在队列的头部，如果要存储数据并且发现数据已经满了，那么便将队列尾部的数据给剔除掉，从而达到我们使用缓存的目的。
这里需要注意一点，队尾存储的数据就是我们最近最少使用的数据，也就是我们在内存满的时候需要剔除掉的数据。

***这里有一个疑问，为什么LruCache内部原理的实现需要用到LinkHashMap来存储数据呐？***

因为LinkHashMap内部是一个数组加双向链表的形式来存储数据，他能够保证插入时候的数据和取出来的时候数据的顺序的一致性。
也就是说，我们以什么样的顺序插入数据，就会以什么样的顺序取出数据。并且更重要的一点是，当我们通过get方法获取数据的时候，这个获取的数据会从队列中跑到队列头来，
从而很好的满足我们LruCache的算法设计思想。


原文链接：https://blog.csdn.net/u013637594/article/details/81866582
