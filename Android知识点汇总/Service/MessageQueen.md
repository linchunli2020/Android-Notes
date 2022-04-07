***handle往messageQueen中发送消息，MessageQueen如何存储消息的：***

MessageQueen存储消息是以单链表的数据接口存储的，通过enqueueMessage方法将消息加入到消息队列中，链表头的消息延迟时间小，链表尾的消息延迟时间大。

handle.postDelay();

链接 ：https://www.jianshu.com/p/f3ff17ccec45
