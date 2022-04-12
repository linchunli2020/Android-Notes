***SharedPreferences  commit 和 apply 方法的区别？***

1、commit 和 apply 虽然都是原子性操作，但是原子的操作不同，commit 是原子提交到数据库，所以从提交数据到存在Disk中都是同步过程，中间不可打断。

2、而 apply 方法的原子操作是原子提交的内存中，而非数据库，所以在提交到内存中时不可打断，之后再异步提交数据到数据库中，因此也不会有相应的返回值。

3、所有commit提交是同步过程，效率会比apply异步提交的速度慢，但是apply没有返回值，永远无法知道存储是否失败。

4、在不关心提交结果是否成功的情况下，优先考虑 apply 方法；

  Commit  有返回值，同步；
  apply   没有返回值，异步；
  
  
  
***SharePreferences特点：***

1.SharePreferences 是线程安全的 里面的方法有大量的synchronized来保障。

2.SharePreferences 不是进程安全的 即使你用了MODE_MULTI_PROCESS 。

3.第一次getSharePreference会读取磁盘文件，异步读取，写入到内存中，后续的getSharePreference都是从内存中拿了。

4.第一次读取完毕之前 所有的get/set请求都会被卡住 等待读取完毕后再执行，所以第一次读取会有ANR风险。

5.所有的get都是从内存中读取。

6.提交都是 写入到内存和磁盘中 。apply跟commit的区别在于

    apply 是内存同步 然后磁盘异步写入任务放到一个单线程队列中 等待调用。方法无返回 即void
    commit 内存同步 只不过要等待磁盘写入结束才返回 直接返回写入成功状态 true or false
    
7.从 Android N 开始, 不再支持 MODE_WORLD_READABLE & MODE_WORLD_WRITEABLE. 一旦指定, 会抛异常 。也不要用MODE_MULTI_PROCESS 迟早被放弃。

8.每次commit/apply都会把全部数据一次性写入到磁盘，即没有增量写入的概念 。 所以单个文件千万不要太大 否则会严重影响性能。

建议用微信的第三方MMKV来替代SharePreference
腾讯有一个MMKV是sharedPreferences的优化版，读取写入性能都有大幅提升，但是没有实现sharedPreferences的registerOnSharedPreferenceChangeListener（）接口，
无法监听值的改变，希望后续能够完善。


***mmkv和SharePreference对比：***

**SharedPreferences存在的问题**

  SP的效率比较低
  1.读写方式：直接I/O
  2.数据格式：xml
  3.写入方式：全量更新

由于SP使用的xml格式保存数据，所以每次更新数据只能全量替换更新数据。
这意味着如果我们有100个数据，如果只更新一项数据，也需要将所有数据转化成xml格式，然后再通过io写入文件中这也导致SP的写入效率比较低

**MMKV 是基于 mmap 内存映射的 key-value 组件，底层序列化/反序列化使用 protobuf 实现，性能高，稳定性强。
从 2015 年中至今在微信上使用，其性能和稳定性经过了时间的验证。近期也已移植到 Android / macOS / Win32 / POSIX 平台，一并开源。**

**MMKV优点**

  1.MMKV实现了SharedPreferences接口，可以无缝切换
  2.通过 mmap 内存映射文件，提供一段可供随时写入的内存块，App 只管往里面写数据，由操作系统负责将内存回写到文件，不必担心 crash 导致数据丢失。
  3.MMKV数据序列化方面选用 protobuf 协议，pb 在性能和空间占用上都有不错的表现
  4.SP是全量更新，MMKV是增量更新，有性能优势。

MMKV加密方式AES CFB-128来加密／解密

我们选择 CFB 而不是常见的 CBC 算法，主要是因为 MMKV 使用 append-only 实现插入/更新操作，流式加密算法更加合适。

AES的工作模式，体现在把明文块加密成密文块的处理过程中。

  ● ECB模式：电码本模式 Electronic Codebook Book（默认）
  ● CBC模式：密码分组链接模式 Cipher Block Chaining
  ● CTR模式：计算器模式 Counter
  ● CFB模式：密码反馈模式 Cipher FeedBack
  ● OFB模式：输出反馈模式 Output FeedBack


链接：https://www.jianshu.com/p/918240c01644

