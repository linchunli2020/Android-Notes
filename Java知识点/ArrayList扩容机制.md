***ArrayList的扩容机制总的来说:***

    ArrayList 的内部实现，其实是用一个对象数组进行存放具体的值，然后用一种扩容的机制，进行数组的动态增长。
    其扩容机制可以理解为，如果元素的个数，大于其容量，则把其容量扩展为原来容量的1.5倍。
    
    
具体参考：https://blog.csdn.net/eases_stone/article/details/79843851
