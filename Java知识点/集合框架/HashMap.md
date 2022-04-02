***HashMap由数组+链表组成，数组是HashMap的主体，链表是为了解决hash冲突而存在的。***

数组是：一个Entry数组（hashmap1.8改为了Node节点），Entry是HashMap的基本单元，每个Entry包含一个key-value键值对，
Entry是HashMap的一个静态内部类，包含key／value／hash值／next（下一个Entry）

Entry数据的存储规则：key的hash值&（length-1）

简单来说，HashMap由数组+链表组成，数组是HashMap的主体，链表是为了解决hash冲突而存在的。如果定位到的数组位置不含链表，那么对于查找／添加操作都很快，仅需一次寻址就可以；
如果定位到的数组位置含有链表，那么对于添加来说，其时间复杂度依然为O（1），因为最新的Entry会插入到链表的头部，仅需要改变引用链即可；
对于查找来说，就需要遍历整个链表了，然后通过key的equals方法逐一比对查找。所以，性能方面考虑，HashMap中的链表出现越少，性能才会越好。

hashmap1.7 https://juejin.cn/post/6844903550917541901

hashmap1.8 https://blog.51cto.com/u_14153136/3116453

***put方法：***

      如果table数组为空时，先创建数组，并设置扩容阀值
      如果key为空时，调用putForNullKey方法进行特殊处理
      计算key的hash值
      根据计算出来的hash值与当前数组的长度计算出在数组中的索引
      先遍历数组索引下的整条链表
      如果该key之前在hashmap中存储了的话，直接替换当前的value值即可；
      如果该key之前没有在hashmap中存储，则进入addEntry方法
      addEntry方法：
      如果当前容量大于或等于容量阀值，就进行扩容
      扩容为原来容量的2倍
      重新计算hash值
      重新得到在新数组中的索引
      创建节点（crreateEntry创建节点方法）
      如果发现Entry是空的，之前没有存值，就直接把值存进去就可以；
      如果Entry有值，即发生了hash碰撞hash冲突，就以单链表头插入的方式存储；
      resize：如何扩容的：
      创建一个新的Entry数组
      将旧Entry数组中的数据复制到新Entry数组中
      将新数组的引用赋给table
      计算新的扩容阀值

***get（Object key）方法：***

      如果key为空，调用getForNullKey进行特殊处理
      获取key对应的Entry（getEntry（key）方法）
      计算key的hash值
      得到数组的索引，然后遍历链表，查看是否有相同key的Entry
      没有的话，返回null

***在java1.8中HashMap的不同：****

      如果链表的长度超过8，则链表将会转化成红黑树；红黑树中节点数目小于6就再转化成链表
      发生hash碰撞时，java1.7会在链表头部插入，而java1.8会在链表尾部插入
      在java1.8中，Entry被Node代替
      
      
***HashMap的Rehash：***

<img width="570" alt="image" src="https://user-images.githubusercontent.com/67937122/161374814-86547b8c-b92c-410f-b545-23f4989d1dcc.png">




      
      
