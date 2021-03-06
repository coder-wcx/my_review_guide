运行时数据区域:
    1.堆
    2.虚拟机栈
        每个栈帧
            局部变量表 存放编译期可知的数据类型、对象引用
            操作数栈 产生的中间结果临时变量
            动态链接 将class文件符号引用转为内存的直接引用
            方法返回地址
    3.本地方法栈
    4.程序计数器
    5.元空间 
    6.直接内存

    GC堆:
        新生代 
            eden 8 
            survivor s0 s1 1 1: 
            每次新生代gc，存活的对象会进入s0/s1 默认15 进入老年代
        老年代
        方法区: 
            永久代 1.8前
            元空间: 使用的是直接内存
    对象创建过程:
        1.类加载检查
        2.分配内存 包括指针碰撞、空闲列表
            在堆内存规整时，使用指针碰撞
            在堆内存不规整时，使用空闲列表
            内存分配并发问题:
                CAS+失败重试:虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性
                TLAB： 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配
        3.初始化零值
        4.设置对象头
            对对象头进行设置，，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 这些信息存放在对象头中。 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。
        5.执行init方法
    对象的内存布局：
        对象头，实例数据，对齐填充
        对象头包含hashcode、gc分代年龄、锁状态，另一部分为类型指针，表明是那个对象的实例
        实例数据存放的就是对象真正的数据
        对齐填充，只是站位