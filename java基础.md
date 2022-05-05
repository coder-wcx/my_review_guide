java基础:
    基本类型的装箱拆箱:
        1.装箱 Integer i = 10; 等价于调用Integer.valueOf();
        2.拆箱 int aa = i; 等价于调用intValue();
    三大特性:
        1.封装
        2.继承
            子类拥有父类所有的方法和属性，包括私有
            子类可以拥有自己的方法和属性
            子类可以用自己的方式实现父类的方法
        3.多态
    浅拷贝深拷贝:
        浅拷贝两个引用指向同一个内存区域
        深拷贝会完全复制整个对象，包括对象内的内部对象
    Object的equal是直接比较对象的内存地址

    String/Stringbuffer/StringBuilder
        String是不可变的 用final修饰
        其余都是继承至ababstractStringBuilder
        StringBuilder非线程安全
        Stringbuffer都用同步保证线程安全

        对String操作时，都会new一个新的对象，而对于StringB而言，不会创建，都是在同一个对象中操作
        private final char value[];因为value[]使用final修饰，且为private，同时并没有暴露方法给外部调用
        String a  = new String("aa");会创建1-2个对象，如果字符串常量池中不存在字符串对象“abc”的引用，那么会在堆中创建 2 个字符串对象“abc”。

        String b = "abc" + "def" 会被优化为 String b = "abcdef"; 常量折叠(Constant Folding)


    final代码块不一定被执行:
        1.当前线程死亡
        2.主动退出 exit(1)
        3.cpu被关闭
    
    值传递:
        方法接受的是实际参数的拷贝
    引用传递:
        方法接受的是实际参数的引用对象的内存地址，对参数做修改会影响到入参

    java只有值传递 按共享传递

    代理模式:
        静态代理:
            对目标对象的每一个方法都手动增强，不灵活，静态代理在编译层面就已经生成具体的class文件了
        动态代理:
            使用代理实现类(cglib动态代理机制)，灵活。在运行时动态生成字节码记载到jvm中
            jdk：
                InvocationHandler: invoke(object,method,args)
                1.通过继承InvocationHandler 实现invoke code对应逻辑
                2.通过Proxy.newProxyInstance(object.getClass.getClassLoader,object.getClass.getInterfaces,InvocationHandler)构建代理对象
                3.通过构造出来的代理对象调用方法，达到增强方法的目的

                但是jdk的动态代理只能对接口进行代理
            
            cglib:
                基于asm的字节码操作库 可以代理未实现任何接口的类
                MethodInterceptor：intercept(object,method,args,proxy);
                Enhancer：
                    // 创建动态代理增强类
                    Enhancer enhancer = new Enhancer();
                    // 设置类加载器
                    enhancer.setClassLoader(clazz.getClassLoader());
                    // 设置被代理类
                    enhancer.setSuperclass(clazz);
                    // 设置方法拦截器
                    enhancer.setCallback(new DebugMethodInterceptor());
                    enhancer.create();

            jdk代理会优于cglib
    
    List:
        ArrayList:
            object[] 
            查询快，插入比较慢
            具有随机访问能力 randomAccess 标记
            扩容1.5倍
        LinkedList:
            双向链表
    Set:
        HashSet：底层是HashMap是hash表 用于不需要保证顺序
        LinkedHashSet: 底层是hash表和链表 元素的插入取出符合fifo 用于需要保证fifo
        TreeSet: 底层是红黑树 元素是有序的 
    Queue:
        queue:单端队列，只能一端插入，一端取出
        deque:双端队列 两头都能操作

        ArrayDeque: 
            底层是基于可变数组和双指针实现的
            支持null
            插入时存在动态扩容的过程，不过整体的时间复杂度为O（1）
        LinkedList: 
            双向链表
            不支持
            虽然 LinkedList 不需要扩容，但是每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。
        PriorityQueue:
            优先队列 总是优先级最高的出队
            利用二叉堆的数据结构 底层是使用可变长的object数组
            通过堆的上浮下层 实现o(logn)的插入删除堆顶操作
    
    hashMap:
        hashMap:
            线程不安全
            支持null的key value
            初始化大小为16 扩容变为原来的2倍
            数组加红黑树
            jdk1.8前:
                数组 + 链表:
                扰动函数需要4次
                拉链法
            jdk1.8后:
                数组 + 链表 + 红黑树:
                链表长度大于8,使用红黑树，数组小于64，先进行数组的扩容
                红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。
        concurrentHashMap:
            jdk1.7:
                使用分段锁:
                    对整个桶数据进行分割分段segment，每一把锁都只会对应一段数据,在多线程访问不同段的数据时，提高了并发度
                segment继承至ReentrantLock HashEntry用于存储数据 
                一个concurrenthashmap包含一个segment数组，一个segment包含一个hashentry,每操作一个hashentry会获取一把锁
            jdk1.8:
                 到了 JDK1.8 的时候已经摒弃了 Segment 的概念，而是直接用 Node 数组+链表+红黑树的数据结构来实现，并发控制使用 synchronized 和 CAS 来操作。
                 采用 CAS 和 synchronized 来保证并发安全
                 synchronized 只锁定当前链表或红黑二叉树的首节点，这样只要 hash 不冲突，就不会产生并发，效率又提升 N 倍。
        hashtable:
            线程安全
            不支持null
            初始化大小为11 扩容为原来的2n +1
        treeMap:
            具有排序功能