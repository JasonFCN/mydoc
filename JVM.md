#### 一：java内存模型

JVM规范，内存共分为虚拟机栈、堆、方法区、程序计数器、本地方法栈五个部分。

![内存模型java7-](JVM.assets/%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8Bjava7-.png)

程序计数器：

​	是一块较小的内存空间，可以看作当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里，字节码解释器工作时就是通过改变这个计数器的值来选择下一条需要执行的字节码指令，分支、跳转、循环、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

​	java虚拟机的多线程是通过轮流切换并分配处理器执行时间的方式来实现的。在一个确定的时刻，一个处理器只能处理一个线程的一条指令。因此，在线程切换时，需要保存各自线程已经运行到的指令位置。也即程序计数器为【线程私有】。

​    当先线程正在执行一个java方法时，该计数器记录的是正在执行的虚拟机字节码执行的地址。如果是Native方法，则该值为【Undefined】。该区域是唯一一个没有任何OutOfMemoryError情况的区域。

java虚拟机栈：

​	线程私有，生命周期与线程相同。它用来描述java方法执行的内存模型。每个方法执行的同时，都会创建一个栈帧【Stack Frame】用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法的调用到执行，都对应着一个栈帧在虚拟机中入栈到出栈的过程。

​    如果线程请求的栈深度大于虚拟机所允许的深度，将抛出StackOverflowError 异常。如果虚拟机栈可以动态扩展（当前大部分的java虚拟机都可以动态扩展，只不过java虚拟机规范中也允许固定长度的虚拟机栈），扩展时无法申请到足够的内存，就会抛出OutOfMemoryError异常。

本地方法栈：

​	与虚拟机栈作用相似，区别在于虚拟机栈为执行java方法服务，本地方法栈为Native方法服务。虚拟机规范并没有强制规定如何实现。甚至Sun HotSpot虚拟机把虚拟机栈与本地方法栈合二为一。同样，本地方法栈也会抛出StackOverflowError 和OutOfMemoryError异常。

Java堆：

​	虚拟机管理的内存中最大的一块。Java堆被所有线程共享，在虚拟机启动时创建。Java堆的唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存。但随着JIT编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化发生，所有对象都在对上分配也渐渐变得不是那么“绝对”了。

​	Java堆是垃圾收集器管理的主要区域，因此也被称作“GC堆”。由于现在收集器基本都是采用分带收集算法，所以Java堆还可以细分为：新生代和老年代；再细致一点的有Eden空间、From Survivor空间、To Survivor空间等。从内存分配角度来看，又可以划分出多个 【线程私有的分配缓冲区】（Thread Local Allocation Buffer, TLAB）。

​	Java虚拟机规范规定，Java堆可以处于物理上不连续的内存空间中，只要逻辑上连续即可。在实现时，既可以实现成固定大小的，也可以是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的。如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

方法区：

​	方法区与Java堆一样，是各线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范吧方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap（非堆），目的应该是与Java堆区分开来。

​	在HotSpot虚拟机实现中，方法区用永久代来实现。目的是能够把分代收集扩展至方法区，这样HotSpot垃圾收集器可以像管理Java堆一样管理该区域。其他虚拟机实现中没有永久代的概念。现在看来，用永久代实现方法区并不是一个好主意，因为这样更容易遇到内存溢出问题，而且有极少数方法（例如String.intern()）会因这个原因导致不同的虚拟机下有不同的表现。在JDK 1.7 中，已经把原本放在永久代的字符串常量池移除。在JDK 1.8中，已经完全移除了永久代的概念。

​	Java虚拟机规范中，对方法区的限制非常宽松，除了可以不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。相对的，垃圾收集在该区域下是比较少见的，但并非数据进入方法区之后就能够“永久”存在了。该区域的内存回收目标主要针对常量池的回收和对类型的卸载，一般来说，该区域的回收“成绩”比较令人满意，尤其是类型的卸载，条件相当苛刻，但是该区域的回收确实是必要的。

​	Java虚拟机规范的规定，当方法区无法免租内存分配需求时，将抛出OutOfMemoryError异常。

运行时常量池：

​	运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表（Constant Pool Table）,用于存放编译器生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。也包括直接引用。

​	运行时常量池相对于Class文件的常量池表的另外一个重要特征是具备动态性，java语言并不要求常量一定只有编译器才能产生，也就是并非预植入Class文件中常量池表的内容才能进入方法区的运行时常量池，运行期也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的intern()方法。

​	当常量池无法申请到内存时会抛出OutOfMemoryError异常。

直接内存：

​	直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域。但也经常被使用，也有可能导致OOM。

​	在NIO类，基于通道（Channel）与缓冲区（Buffer）的I/O方式，使用了Native函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。

​	虽然直接内存不受Java堆的限制，但是还是会受本机总内存大小以及处理器寻址空间的限制。

#### 二：HotSpot虚拟机对象

Java堆上的对象是如何进行创建、布局、访问的？

创建：

 1. 判断该类是否已加载？

    如果未加载，则先执行相应的类加载过程

 2. 分配内存

    ​	如果内存是规整的，则使用“指针碰撞”（Bump the Pointer）的方式，分配内存空间。

    如果内存不是规整的，则使用“空闲列表”（Free List）的方式，分配内存空间。

    内存是否规整，跟垃圾收集器有关。如Serial、ParNew等收集器采用“指针碰撞”的方式，而CMS通常采用空间列表方式。

    ​	保证并发安全：1. CAS 2. TLAB

    ​	设置零值

 3. 设置对象头

    ​	设置类型元信息、对象的哈希码、对象的GC分代年龄等

 4. 对象初始化

对象布局：

​	对象在内存中分为3个区域：对象头（Header）、实例数据（Instance Data）、对齐填充（Padding）.

​	对象头：

​		运行时数据（Mark Word）：

​			记录哈希码、GC分代年龄、锁状态标志、lock Record、偏向线程ID、偏向时间戳等信息，数据长度32位和64位虚拟机上分别为32bit和64bit（未开启压缩指针）。

​	![hotspotMarkWord](JVM.assets/hotspotMarkWord.png)

​		类型指针（Class Pointer）：

​			指向对象的类元数据的指针，来确定该对象属于哪个类型。但并不是所有的虚拟机实现都必须保留类型指针。

​		数组长度（当对象类型为数组时）：

​			普通Java对象实例可以通过对象的元数据信息确定对象大小，但数组并不能确定数组的大小。所以需要单独维护长度大小。

​	实例数据：

​		记录该对象的字段（包含父类）。字段的存储顺序受虚拟机分配策略参数（FieldsAllocationStyle）和字段在Java源码中定义的顺序影响。

​		默认的分配策略为：longs/doubles、ints、shorts/chars、bytes/booleans、oops（Ordinary Object Pointers）。

​		在默认分配策略下，父类字段会出现在子类之前。如果CompactFields参数值为true，那么子类中较窄的字段有可能插入到父类字段的空隙之中。

​	对齐填充：

​		虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，对象头是8字节的倍数，就要求在实例数据不够8字节的整倍数	时，需要填充字节。

对象访问：

​	对象访问通过栈上的reference来操作堆上的对象。

​	reference的实现方案有两种：句柄方式和直接指针两种。

​	句柄：内存中划分出一块内存作为句柄池。reference中存储的是对象的句柄地址。句柄中包含对象实例数据和对象类型数据各自的具体地址信息。优点：对象移动后，不需要改变reference。

​	直接指针：reference中存储的是对象的实际地址。对象实例的对象头中存储着对象类型数据的地址。优点：访问速度快。（HotSpot实现）

#### 三：垃圾收集算法与收集器

​	**垃圾收集算法：**

​	哪些内存需要回收？换句话说，哪些对象是可以被回收的？

​	判断一个对象是否存活，有两种算法：

   - **引用计数算法**

     给对象添加一个引用计数器，每当有一个地方引用它时，就加1；当引用失效时，就减1。

     但会产生循环引用问题。

   - **可达性分析算法**

     从“Root GC”对象开始，向下搜索所有对象，如果一个对象没有与根节点相连，则证明该对象不可用。也就被判定为可回收的对象。

     Root GC：

      - 虚拟机栈中引用的对象
      - 方法区中类静态属性引用的对象
      - 方法区中常量引用的对象
      - 本地方法栈中JNI（Native方法）引用的对象

     **引用类型**：

     ​	在JDK1.2以前，引用的定义：如果reference类型的数据中存储的数值代表的是另外一块内存的起始地址，就称这块内存代表着一个引用。这样的定义让一个引用只有两个状态：被引用和没有被引用。如果想表达一个对象可有可无的状态，就无能为力了。

     ​	在JDK1.2之后，对引用进行了扩充，分为4种：强引用、软引用、弱引用、虚引用。

      - 强引用（Strong Reference）

        ​	强引用到处存在，类似“Object obj = new Object()”这类的引用，只要强引用存在，对象就永远不会被垃圾收集器回收。

      - 软引用（Soft Reference）

        ​	软引用用来描述一些还有用但并非必要的对象。在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。

      - 弱引用（Weak Reference）

        弱引用来描述非必要对象的。它比软引用更弱一些。被软引用关联的对象只能生存到下次垃圾收集之前。当垃圾收集器工作时，无论内存是否足够，都会回收掉被弱引用关联的对象。

      - 虚引用（Phantom Reference）

        ​	最弱的引用，一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。

     **finalize()方法**：

     ​	当一个对象不可达时，也并不是说该对象就是一定要回收的。当该对象不可达时，会被第一次标记，并进行一次筛选，筛选条件是是否有必要执行finalize()方法。当对象没有覆盖finalize()方法，或者finalize()方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。

     ​	当这个对象被判定为有必要执行finalize()方法，那么这个对象将放置到一个叫做F-Queue的队列中，并在稍后有一个由虚拟机自动建立的、低优先级的Finalizer线程去执行它。这里的执行是指虚拟机会触发这个方法，但并不承诺会等待它运行结束，原因是：如果一个对象在finalize()方法中执行缓慢，或者发生了死循环，将很可能导致F-Queue队列中其他对象永久处于等待，甚至导致整个内存回收系统崩溃。稍后，会对队列中的对象进行第二次标记，那些已经重新关联跟对象的对象会被移出队列。如果对象在第二次标记后依然不可达，基本被认为被回收了。

     **回收方法区**

     新生代的回收效率一般为70% ~ 95%，而方法区的回收效率要低很多。

     方法区中的常量池，回收字面量很简单，如果没有任何引用引用该常量，当发生内存回收时，如果有必要，该常量就会被系统清理出常量池。常量池中的类、接口、方法、字段的符号引用也类似。

     一个类是否可以被回收，条件是相对苛刻许多。要同时满足如下三个条件：

     - 该类所有实例都已经被回收
     - 加载该类的类加载器已经被回收
     - 该类的类对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

     当同时满足了上述三条条件，也只能说“可以”，而不会说和对象一样，不使用了就必然会回收。

     在大量使用反射、动态代理、CGLib等ByteCode框架，动态生成JSP以及OSGi这类别烦自定义ClassLoader的场景都需要虚拟机具备卸载类的功能。

     **垃圾收集算法**

      - 标记-清除算法（Mark-Sweep）

        ​	分为标记和清除两个阶段：首先标记出需要回收的对象，在标记完成后统一回收所有被标记的对象。标记过程在介绍finalize()方法时已经介绍过了。该算法是最基础的算法，之后的算法也都是针对该算法的不足并行改进的。主要不足有两点：一个是效率问题，两个过程效率都不高；另一个是空间问题，标记清除后会产生大量不连续的内存碎片，空间碎片太多可能会导致以后在程序运行中需要分配较大对象是，无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

      - 复制算法（Copying）

        ​	将内存按容量划分为大小相等的两块，每次只是用一块，当一块用完了，就把存活的对象复制到另一块内存上，然后把已用过的内存空间一次清理掉。对象创建时分配空间，直接移动指针，顺序分配即可。

        ​	该算法主要应用于新生代的垃圾回收，新生代的对象98%是“朝生夕死”的。而是将内存分为一块较大的Eden空间和两块较小的Survivor空间，每次只是用Eden空间和其中一块Survivor空间，当回收时，将Eden和Survivor存活的对象一次性地复制到两一块Survivor区，最后清理掉刚使用过的Eden和Survivor。HotSpot虚拟机默认Eden ：Sruvivor=8 ：1，也就意味着系统默认新生代的可利用的内存占比为：90%。每次回收时，我们不能保证只有少于10%的对象存活，因此需要依赖其他内存（老年代）进行分配担保（Handle Promotion）。

      - 标记-整理算法（Mark-Compact）

        ​	对标记后存活的对象向一段移动，然后清理掉边界以外的内存。

      - 分代收集算法（Generational Collection）

        ​	根据对象存货周期的不同将内存划分为几块。再根据各内存区域的特点采用适当的收集算法。一般把Java堆分为新生代和老年代。新生代采用【复制算法】，老年代采用【标记-清理】或者【标记-整理】算法进行回收。

     **HotSpot的算法实现**

     ​	枚举根节点：

     ​		根节点存在于全局性的引用以及执行上下文（栈帧中的本地变量表）中，有的应用仅仅是方法区都有数百兆。另外，在GC停顿上更要求可达性分析过程不能太长时间。因为要保证分析结果的一致性，必须在分析过程中不能让对象关系还在变化。GC进行时，必须停顿所有Java执行线程（Stop The World）。

     ​		当前主流的Java虚拟机使用的都是准确式GC（虚拟机要能判断一个引用（32bit的整数）到底是引用还是整数，准确式虚拟机采用句柄方式维护引用的稳定性）。HotSpot的实现中，采用一组称为OopMap的数据结构来维护引用，在类加载完成后，虚拟机吧对象内什么偏移量上是什么类型的数据计算出来，在JIT编译过程中，也会在特定的位置记录下栈和寄存器中哪些位置是引用。

     ​	安全点：

     ​		为每条指令生成OopMap是不现实的。HotSpot只会在‘特定位置’记录这些信息，这个“特定位置”称为安全点（Safepoint）,即程序执行时并非在所有地方都能停顿下来开始GC，只有在到达安全点时才能暂停。安全点的选定既不能太少以致于让GC等待时间太长，也不能过于频繁以致于过分增大运行时的负载。原则是：是否具有让程序长时间执行的特征。“长时间执行”是指指令序列复用，例如方法调用、循环跳转、异常跳转等。

     ​		另一个问题，如何在GC发生时，让所有线程（不包含执行JIT调用的线程）到跑到最近的安全点再停顿下来。两种方案：抢先式中断（Preemptive Suspension）和主动式中断（Voluntary Suspension）。抢先式中断是把所有线程全部中断，如果有不在安全点上的线程，就恢复线程。几乎没有虚拟机实现该方案。主动式中断是在GC发生时，只设置一个标志，各线程主动的轮询这个标志，标志为真时，线程中断挂起。

     ​	安全区域：

     ​		正在执行的线程能够主动轮询标志，中断挂起。那如果是休眠状态或者阻塞状态的线程，怎么响应JVM的中断请求呢？采用安全区域（Safe Region）来解决。安全区域是指在一段代码中，引用关系不会发生变化的区域。该区域的任意地方都是可以安全开始GC的。

     ​		在线程执行到安全区域时，首先标识自己已经进入安全区域，当在这段时间里发生GC,不用管是处于安全区域的线程，它继续执行，当该线程要离开安全区域时，它要检查系统是否已经完成了根节点枚举（或整个GC过程），如果完成，则继续执行，否则，它必须等待直到收到可以离开安全区域的信号为止。

     **垃圾收集器：**

     ​	Serial 收集器：

     ​		单线程新生代收集器，在进行垃圾收集时，会停掉其他所有线程，直到收集结束。看似鸡肋的收集器，但它仍是虚拟机运行在Client模式下的默认新生代收集器。它简单而高效（与其他收集器的单线程相比）。它是单CPU环境下，拥有很高的收集效率。在桌面应用场景中，是一个很好的选择。

     ​	ParNew 收集器：

     ​		多线程版本的Serial收集器。运行在Service模式下的默认新生代并行垃圾收集器。除了Serial收集器以外，它是能与CMS收集器相配合的唯一收集器。

     注意：

     ​	并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。

     ​	并发（Concurrent）：值用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），用户程序在继续执行，而垃圾收集程序运行于另一个CPU上。

     ​	Parallel Scavenge 收集器：

     ​		采用复制算法，运行在新生代的并行的多线程收集器，关注点在于吞吐量（Throughput）。吞吐量 = 运行用户代码时间  / （运行用户代码时间 + 垃圾收集时间）。
     
     ​		高吞吐量可以高效率利用CPU时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。低停顿时间适合需要与用户交互的程序，良好的响应速度能提升用户体验。
     
     ​		相关参数：
     
     ​			-XX:MaxGCPauseMillis：最大垃圾收集时间。大于0的值。设置过小不会使系统的垃圾收集速度变得更快，GC停顿时间缩短是以牺牲吞吐量和新生代空间来换取的。
     
     ​			-XX:GCTimeRotio：设置吞吐量的大小。大于0小于100的整数。若该值为19，则表示垃圾收集耗时为1，用户运行时间为19，吞吐量为19/（1+19）= 95%。
     
     ​			-XX:+UseAdaptiveSizePolicy：开启GC自适应调节策略。这也是该收集器与Parnew收集器的一个重要区别。
     
     ​	Serial Old 收集器：
     
     ​		Serial收集器的老年代版本，也是单线程收集器。从用【标记-整理】算法。主要是给Client模式下的虚拟机使用。在Server模式下，一是在JDK1.5及以前版本中与Parallel Scavenge收集器搭配使用（Parallel Scavenge收集器架构中本身有PS MarkSweep收集器来进行老年代收集，但功能与Serial Old相似）；二是作为CMS收集器的后备方案，在并发收集发生Concurrent Mode Failure时使用。
     
     ​	Parallel Old 收集器：
     
     ​		Parallel Scavenge的老年代版本，使用多线程和【标记-整理】算法。在JDK1.6中开始提供。在此之前，新生代的Parallel Scavenge一直没有一个“好队友”与之配合工作。老年代的收集工作老是拖后腿。
     
     ​		有了Parallel Old收集器搭配，在高吞吐量已经CPU资源敏感场景中，是一个非常好的选择。
     
     ​	CMS 收集器：
     
     ​		以最短回收停顿时间为目标的收集器。非常适合互联网站或者B/S系统的服务端上，响应快，低停顿。CMS收集器基于【标记-清除】算法，运作过程分为：
     
      - 初始标记（CMS initial mark）
     
        STW，标记根对象能关联的对象，速度快
     
      - 并发标记（CMS concurrent mark）
     
        进行GC Roots Tracing
     
      - 重新标记（CMS remark）
     
        修正并发标记期间标记变动的那一部分对象的标记记录，STW，停顿时间一般比初始标记阶段稍长一些，但远比并发标记时间短。
     
      - 并发清除（CMS concurrent sweep）
     
        与用户线程并发执行，清除垃圾对象
     
        ----------------------------
     
        优点：并发，低停顿
     
        缺点：
     
        - CPU资源敏感。CMS默认启动的回收线程数是（CPU数量 + 3）/ 4，当CPU数量为2时，垃圾收集线程就占了一半，大大影响程序运行效率。
        - CMS收集器无法处理浮动垃圾（Floaing Garbage），可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生。浮动垃圾：由于CMS并发清除阶段用户线程还在运行，伴随程序运行自然就会有新的垃圾产生，这一部分垃圾出现在标记过程之后，CMS无法在当次收集中处理掉它们，只好留在下一次GC时再清理掉。这部分垃圾就是浮动垃圾。又由于垃圾收集阶段用户线程还在运行，也就需要预留有足够的内存空间给用户线程使用，因此CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，需要预留一部分空间提供并发收集时的程序运作使用。适当调高参数-XX:CMSInitiatingOccupancyFraction的值来提高出发百分比，以便降低内存回收次数从而获取更好的性能。要是CMS运行期间预留的内存无法满足程序需要，就会出现一次“Concurrent Mode Failure”失败，这时虚拟机将启动后备预案：临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间更长了。所以-XX:CMSInitiatingOccupancyFraction参数设置太高很容易导致大量“Concurrent Mode Failure”失败，降低性能。
        - 产生大量内存碎片。空间碎片过多时，在分配大对象时找不到足够大的空间，不得不提前触发一次Full GC。参数-XX:+UseCMSCompactAtFullCollection（默认开启），用于在CMS收集器顶不住要进行FullGC时开启内存碎片的合并整理过程，该过程是无法并发的，空间碎片问题解决了，但停顿时间不得不变长。-XX:CMSFullGCsBeforeCompaction用于设置执行多少次不压缩的Full GC后，跟着来一次带压缩的（默认0，表示每次进入Full GC时都进行碎片整理）。
     
     ​	G1 收集器：
     
     ​		面向服务端的收集器。
     
     ​		特点：
     
        - 并行与并发
     
          充分利用多核CPU，缩短STW时间
     
        - 分代收集
     
          保留分代思想，不需要其他收集器配合。
     
        - 空间整合
     
          从整体来看是基于【标记-整理】算法，局部来看是基于【复制】算法。运行期间不会产生内存碎片，分配大对象不会因为无法找到连续的内存空间而提前触发一次GC。
     
        - 可预测的停顿
     
          除了低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒，几乎做到实时Java的垃圾收集器的特征了。
     
          ------------
     
          G1将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代与老年代不再是物理隔离的了。它们都是一部分Region（不需要连续）的集合。
     
          G1的可预测的停顿是基于它有计划的避免在整个堆中进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆积的价值大小，在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region。
     
          分区域的思路看似简单，其实不然。G1从实验到商用，用了近10年。G1的垃圾收集过程做不到以Region为单位进行收集。因为一个对象分配到某个Region中，它并非只能本Region中的其他对象引用。在可达性分析中，岂不是要扫描全表来保证准确性？
     
          在G1中，每个Region都对应着一个Remembered Set，虚拟机发现程序在对Reference类型的数据进行写操作时，会产生一个Write Barrier暂时中断写操作，检查Reference引用的对象是否处于不同的Region之中（在分代的例子中，就是检查是否老年代中的对象引用了新生代中的对象），如果是，便通过CardTable把相关引用信息记录到被引用对象所属的Region的Remembered Set之中。当进行内存回收时，在GC根节点的枚举范围中加入Remembered Set即可保证不对全堆扫描也不会有遗漏。
     
          G1收集器的运作过程：
     
           - 初始标记（Initial Marking）
     
             标记一下被GC Roots关联的对象，并修改TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的Region中创建新对象，该阶段需要停顿，但耗时很短。
     
           - 并发标记（Concurrent Marking）
     
             从GC Roots 开始，进行可达性分析，耗时较长，与用户线程并发执行。
     
           - 最终标记（Final Marking）
     
             修正在并发标记阶段因用户线程继续运行而导致标记产生变动的那一部分标记记录，虚拟机将这阶段的变化记录在线程Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这阶段需要停顿，但可并行执行。
     
           - 筛选回收（Live Data Counting and Evacuation）
     
             首先对各个Region的回收价值排序，根据用户设置的期望指定计划。
     
     ​	GC日志：
     
     ```console
     33.125: [GC [DefNew: 3324k->152k(3712k), 0.0025925 secs] 3324k->152k(11904k), 0.0031680 secs]
     100.667: [Full GC [Tenured: 0k->210k(10240k), 0.0149142 secs] 4603k->210k(19456k), [Perm : 2999k->2999k(21248k)], 0.0150007 secs] [Times: user=0.01 sys=0.00 real=0.02 secs]
     ```
     
     ​	最前面的数字：33.125 和 100.667代表GC发生的时间，该数值是从Java虚拟机启动以来经过的秒数。
     
     ​	GC日志开头的 [GC 和 [Full GC说明这次收集的停顿类型，而不是用来区分新生代GC还是老年代GC的。如果有“Full",说明这次GC是发生了STW的。
     
     ​	[DefNew [Tenured [Perm 表示发生的区域，这里显示的区域名称与使用的GC收集器密切相关。例如：Serial收集器中的新生代名为Default New Generation 则日志显示为 [DefNew；如果是ParNew收集器，新生代名称为Parallel New Generation 日志显示为[ParNew；如果是Parallel Scavenge收集器，日志显示为PSYoungGen。
     
     ​	3324k->152k(3712k) 表示：GC前该内存区域已使用容量->GC后该区域已使用容量(该内存区域总容量)。
     
     ​	方括号外3324k->152k(11904k)表示：GC前Java堆已使用容量->GC后Java堆已使用容量(Java堆总容量)。
     
     ​	0.0025925 secs 表示：GC所占用的时间（秒）
     
     ​	[Times: user=0.01 sys=0.00 real=0.02 secs] 表示：用户态消耗的CPU时间、内核态消耗的CPU时间、操作从开始到结束所经过的墙钟时间（Wall Clock Time）。墙钟时间包括非运算时间，如等待磁盘IO、线程阻塞，而CPU时间不包括这些。在多核或者多CPU情况下，多线程会叠加这些CPU时间，所以user,sys时间超过real属于正常。
     
     Java参数：
     
     - ##### 标准参数
     
     ```console
     java -help
     	java [-options] class [args...]
                (执行类)
     或  java [-options] -jar jarfile [args...]
                (执行 jar 文件)
     其中选项包括:
         -d32          使用 32 位数据模型 (如果可用)
         -d64          使用 64 位数据模型 (如果可用)
         -server       选择 "server" VM
                       默认 VM 是 server.
     
         -cp <目录和 zip/jar 文件的类搜索路径>
         -classpath <目录和 zip/jar 文件的类搜索路径>
                       用 ; 分隔的目录, JAR 档案
                       和 ZIP 档案列表, 用于搜索类文件。
         -D<名称>=<值>
                       设置系统属性
         -verbose:[class|gc|jni]
                       启用详细输出
         -version      输出产品版本并退出
         -version:<值>
                       警告: 此功能已过时, 将在
                       未来发行版中删除。
                       需要指定的版本才能运行
         -showversion  输出产品版本并继续
         -jre-restrict-search | -no-jre-restrict-search
                       警告: 此功能已过时, 将在
                       未来发行版中删除。
                       在版本搜索中包括/排除用户专用 JRE
         -? -help      输出此帮助消息
         -X            输出非标准选项的帮助
         -ea[:<packagename>...|:<classname>]
         -enableassertions[:<packagename>...|:<classname>]
                       按指定的粒度启用断言
         -da[:<packagename>...|:<classname>]
         -disableassertions[:<packagename>...|:<classname>]
                       禁用具有指定粒度的断言
         -esa | -enablesystemassertions
                       启用系统断言
         -dsa | -disablesystemassertions
                       禁用系统断言
         -agentlib:<libname>[=<选项>]
                       加载本机代理库 <libname>, 例如 -agentlib:hprof
                       另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
         -agentpath:<pathname>[=<选项>]
                       按完整路径名加载本机代理库
         -javaagent:<jarpath>[=<选项>]
                       加载 Java 编程语言代理, 请参阅 java.lang.instrument
         -splash:<imagepath>
                       使用指定的图像显示启动屏幕
     ```
     
     - ##### X参数：非标准参数
     
     ```
     java -X
     -Xmixed           混合模式执行 (默认)
     -Xint             仅解释模式执行
     -Xbootclasspath:<用 ; 分隔的目录和 zip/jar 文件>
                       设置搜索路径以引导类和资源
     -Xbootclasspath/a:<用 ; 分隔的目录和 zip/jar 文件>
                       附加在引导类路径末尾
     -Xbootclasspath/p:<用 ; 分隔的目录和 zip/jar 文件>
                       置于引导类路径之前
     -Xdiag            显示附加诊断消息
     -Xnoclassgc       禁用类垃圾收集
     -Xincgc           启用增量垃圾收集
     -Xloggc:<file>    将 GC 状态记录在文件中 (带时间戳)
     -Xbatch           禁用后台编译
     -Xms<size>        设置初始 Java 堆大小
     -Xmx<size>        设置最大 Java 堆大小
     -Xss<size>        设置 Java 线程堆栈大小
     -Xprof            输出 cpu 配置文件数据
     -Xfuture          启用最严格的检查, 预期将来的默认值
     -Xrs              减少 Java/VM 对操作系统信号的使用 (请参阅文档)
     -Xcheck:jni       对 JNI 函数执行其他检查
     -Xshare:off       不尝试使用共享类数据
     -Xshare:auto      在可能的情况下使用共享类数据 (默认)
     -Xshare:on        要求使用共享类数据, 否则将失败。
     -XshowSettings    显示所有设置并继续
     -XshowSettings:all
                       显示所有设置并继续
     -XshowSettings:vm 显示所有与 vm 相关的设置并继续
     -XshowSettings:properties
                       显示所有属性设置并继续
     -XshowSettings:locale
                       显示所有与区域设置相关的设置并继续
     ```
     
     - ##### XX参数：
     
     ```console
     # 查看工程学选择的JVM参数以及堆空间大小和选定的垃圾收集器。
     D:\IdeaProjects\my01>java -XX:+PrintCommandLineFlags -version
     -XX:InitialHeapSize=258687808 -XX:MaxHeapSize=4139004928 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+U
     seParallelGC
     java version "1.8.0_251"
     Java(TM) SE Runtime Environment (build 1.8.0_251-b08)
     Java HotSpot(TM) 64-Bit Server VM (build 25.251-b08, mixed mode)
     
     # 查看XX参数默认值
     D:\IdeaProjects\my01>java -XX:+PrintFlagsInitial -version
     [Global flags]
          intx ActiveProcessorCount                      = -1                                  {product}
         uintx AdaptiveSizeDecrementScaleFactor          = 4                                   {product}
         uintx AdaptiveSizeMajorGCDecayTimeScale         = 10                                  {product}
         uintx AdaptiveSizePausePolicy                   = 0                                   {product}
         uintx AdaptiveSizePolicyCollectionCostMargin    = 50                                  {product}
         uintx AdaptiveSizePolicyInitializingSteps       = 20                                  {product}
         uintx AdaptiveSizePolicyOutputInterval          = 0                                   {product}
         uintx AdaptiveSizePolicyWeight                  = 10                                  {product}
         uintx AdaptiveSizeThroughPutPolicy              = 0                                   {product}
         uintx AdaptiveTimeWeight                        = 25                                  {product}
          bool AdjustConcurrency                         = false                               {product}
          bool AggressiveHeap                            = false                               {product}
          bool AggressiveOpts                            = false                               {product}
          intx AliasLevel                                = 3                                   {C2 product}
          bool AlignVector                               = true                                {C2 product}
          intx AllocateInstancePrefetchLines             = 1                                   {product}
          intx AllocatePrefetchDistance                  = -1                                  {product}
          intx AllocatePrefetchInstr                     = 0                                   {product}
          intx AllocatePrefetchLines                     = 3                                   {product}
          intx AllocatePrefetchStepSize                  = 16                                  {product}
          intx AllocatePrefetchStyle                     = 1                                   {product}
          bool AllowJNIEnvProxy                          = false                               {product}
          bool AllowNonVirtualCalls                      = false                               {product}
          bool AllowParallelDefineClass                  = false                               {product}
          bool AllowUserSignalHandlers                   = false                               {product}
          bool AlwaysActAsServerClassMachine             = false                               {product}
          bool AlwaysCompileLoopMethods                  = false                               {product}
          bool AlwaysLockClassLoader                     = false                               {product}
          bool AlwaysPreTouch                            = false                               {product}
          bool AlwaysRestoreFPU                          = false                               {product}
          bool AlwaysTenure                              = false                               {product}
          bool AssertOnSuspendWaitFailure                = false                               {product}
          bool AssumeMP                                  = false                               {product}
          intx AutoBoxCacheMax                           = 128                                 {C2 product}
         uintx AutoGCSelectPauseMillis                   = 5000                                {product}
          intx BCEATraceLevel                            = 0                                   {product}
          intx BackEdgeThreshold                         = 100000                              {pd product}
          bool BackgroundCompilation                     = true                                {pd product}
         uintx BaseFootPrintEstimate                     = 268435456                           {product}
          intx BiasedLockingBulkRebiasThreshold          = 20                                  {product}
          intx BiasedLockingBulkRevokeThreshold          = 40                                  {product}
          intx BiasedLockingDecayTime                    = 25000                               {product}
          intx BiasedLockingStartupDelay                 = 4000                                {product}
          bool BindGCTaskThreadsToCPUs                   = false                               {product}
          bool BlockLayoutByFrequency                    = true                                {C2 product}
          intx BlockLayoutMinDiamondPercentage           = 20                                  {C2 product}
          bool BlockLayoutRotateLoops                    = true                                {C2 product}
          bool BranchOnRegister                          = false                               {C2 product}
          bool BytecodeVerificationLocal                 = false                               {product}
          bool BytecodeVerificationRemote                = true                                {product}
          bool C1OptimizeVirtualCallProfiling            = true                                {C1 product}
          bool C1ProfileBranches                         = true                                {C1 product}
          bool C1ProfileCalls                            = true                                {C1 product}
          bool C1ProfileCheckcasts                       = true                                {C1 product}
          bool C1ProfileInlinedCalls                     = true                                {C1 product}
          bool C1ProfileVirtualCalls                     = true                                {C1 product}
          bool C1UpdateMethodData                        = true                                {C1 product}
          intx CICompilerCount                           = 2                                   {product}
          bool CICompilerCountPerCPU                     = false                               {product}
          bool CITime                                    = false                               {product}
          bool CMSAbortSemantics                         = false                               {product}
         uintx CMSAbortablePrecleanMinWorkPerIteration   = 100                                 {product}
          intx CMSAbortablePrecleanWaitMillis            = 100                                 {manageable}
         uintx CMSBitMapYieldQuantum                     = 10485760                            {product}
         uintx CMSBootstrapOccupancy                     = 50                                  {product}
          bool CMSClassUnloadingEnabled                  = true                                {product}
         uintx CMSClassUnloadingMaxInterval              = 0                                   {product}
          bool CMSCleanOnEnter                           = true                                {product}
          bool CMSCompactWhenClearAllSoftRefs            = true                                {product}
         uintx CMSConcMarkMultiple                       = 32                                  {product}
          bool CMSConcurrentMTEnabled                    = true                                {product}
         uintx CMSCoordinatorYieldSleepCount             = 10                                  {product}
          bool CMSDumpAtPromotionFailure                 = false                               {product}
          bool CMSEdenChunksRecordAlways                 = true                                {product}
         uintx CMSExpAvgFactor                           = 50                                  {product}
          bool CMSExtrapolateSweep                       = false                               {product}
         uintx CMSFullGCsBeforeCompaction                = 0                                   {product}
         uintx CMSIncrementalDutyCycle                   = 10                                  {product}
         uintx CMSIncrementalDutyCycleMin                = 0                                   {product}
          bool CMSIncrementalMode                        = false                               {product}
         uintx CMSIncrementalOffset                      = 0                                   {product}
          bool CMSIncrementalPacing                      = true                                {product}
         uintx CMSIncrementalSafetyFactor                = 10                                  {product}
         uintx CMSIndexedFreeListReplenish               = 4                                   {product}
          intx CMSInitiatingOccupancyFraction            = -1                                  {product}
         uintx CMSIsTooFullPercentage                    = 98                                  {product}
        double CMSLargeCoalSurplusPercent                = 0.950000                            {product}
        double CMSLargeSplitSurplusPercent               = 1.000000                            {product}
          bool CMSLoopWarn                               = false                               {product}
         uintx CMSMaxAbortablePrecleanLoops              = 0                                   {product}
          intx CMSMaxAbortablePrecleanTime               = 5000                                {product}
         uintx CMSOldPLABMax                             = 1024                                {product}
         uintx CMSOldPLABMin                             = 16                                  {product}
         uintx CMSOldPLABNumRefills                      = 4                                   {product}
         uintx CMSOldPLABReactivityFactor                = 2                                   {product}
          bool CMSOldPLABResizeQuicker                   = false                               {product}
         uintx CMSOldPLABToleranceFactor                 = 4                                   {product}
          bool CMSPLABRecordAlways                       = true                                {product}
         uintx CMSParPromoteBlocksToClaim                = 16                                  {product}
          bool CMSParallelInitialMarkEnabled             = true                                {product}
          bool CMSParallelRemarkEnabled                  = true                                {product}
          bool CMSParallelSurvivorRemarkEnabled          = true                                {product}
         uintx CMSPrecleanDenominator                    = 3                                   {product}
         uintx CMSPrecleanIter                           = 3                                   {product}
         uintx CMSPrecleanNumerator                      = 2                                   {product}
          bool CMSPrecleanRefLists1                      = true                                {product}
          bool CMSPrecleanRefLists2                      = false                               {product}
          bool CMSPrecleanSurvivors1                     = false                               {product}
          bool CMSPrecleanSurvivors2                     = true                                {product}
         uintx CMSPrecleanThreshold                      = 1000                                {product}
          bool CMSPrecleaningEnabled                     = true                                {product}
          bool CMSPrintChunksInDump                      = false                               {product}
          bool CMSPrintEdenSurvivorChunks                = false                               {product}
          bool CMSPrintObjectsInDump                     = false                               {product}
         uintx CMSRemarkVerifyVariant                    = 1                                   {product}
          bool CMSReplenishIntermediate                  = true                                {product}
         uintx CMSRescanMultiple                         = 32                                  {product}
         uintx CMSSamplingGrain                          = 16384                               {product}
          bool CMSScavengeBeforeRemark                   = false                               {product}
         uintx CMSScheduleRemarkEdenPenetration          = 50                                  {product}
         uintx CMSScheduleRemarkEdenSizeThreshold        = 2097152                             {product}
         uintx CMSScheduleRemarkSamplingRatio            = 5                                   {product}
        double CMSSmallCoalSurplusPercent                = 1.050000                            {product}
        double CMSSmallSplitSurplusPercent               = 1.100000                            {product}
          bool CMSSplitIndexedFreeListBlocks             = true                                {product}
          intx CMSTriggerInterval                        = -1                                  {manageable}
         uintx CMSTriggerRatio                           = 80                                  {product}
          intx CMSWaitDuration                           = 2000                                {manageable}
         uintx CMSWorkQueueDrainThreshold                = 10                                  {product}
          bool CMSYield                                  = true                                {product}
         uintx CMSYieldSleepCount                        = 0                                   {product}
         uintx CMSYoungGenPerWorker                      = 67108864                            {pd product}
         uintx CMS_FLSPadding                            = 1                                   {product}
         uintx CMS_FLSWeight                             = 75                                  {product}
         uintx CMS_SweepPadding                          = 1                                   {product}
         uintx CMS_SweepTimerThresholdMillis             = 10                                  {product}
         uintx CMS_SweepWeight                           = 75                                  {product}
          bool CheckEndorsedAndExtDirs                   = false                               {product}
          bool CheckJNICalls                             = false                               {product}
          bool ClassUnloading                            = true                                {product}
          bool ClassUnloadingWithConcurrentMark          = true                                {product}
          intx ClearFPUAtPark                            = 0                                   {product}
          bool ClipInlining                              = true                                {product}
         uintx CodeCacheExpansionSize                    = 65536                               {pd product}
         uintx CodeCacheMinimumFreeSpace                 = 512000                              {product}
          bool CollectGen0First                          = false                               {product}
          bool CompactFields                             = true                                {product}
          intx CompilationPolicyChoice                   = 0                                   {product}
     ccstrlist CompileCommand                            =                                     {product}
         ccstr CompileCommandFile                        =                                     {product}
     ccstrlist CompileOnly                               =                                     {product}
          intx CompileThreshold                          = 10000                               {pd product}
          bool CompilerThreadHintNoPreempt               = true                                {product}
          intx CompilerThreadPriority                    = -1                                  {product}
          intx CompilerThreadStackSize                   = 0                                   {pd product}
         uintx CompressedClassSpaceSize                  = 1073741824                          {product}
         uintx ConcGCThreads                             = 0                                   {product}
          intx ConditionalMoveLimit                      = 3                                   {C2 pd product}
          intx ContendedPaddingWidth                     = 128                                 {product}
          bool ConvertSleepToYield                       = true                                {pd product}
          bool ConvertYieldToSleep                       = false                               {product}
          bool CrashOnOutOfMemoryError                   = false                               {product}
          bool CreateMinidumpOnCrash                     = false                               {product}
          bool CriticalJNINatives                        = true                                {product}
          bool DTraceAllocProbes                         = false                               {product}
          bool DTraceMethodProbes                        = false                               {product}
          bool DTraceMonitorProbes                       = false                               {product}
          bool Debugging                                 = false                               {product}
         uintx DefaultMaxRAMFraction                     = 4                                   {product}
          intx DefaultThreadPriority                     = -1                                  {product}
          intx DeferPollingPageLoopCount                 = -1                                  {product}
          intx DeferThrSuspendLoopCount                  = 4000                                {product}
          bool DeoptimizeRandom                          = false                               {product}
          bool DisableAttachMechanism                    = false                               {product}
          bool DisableExplicitGC                         = false                               {product}
          bool DisplayVMOutputToStderr                   = false                               {product}
          bool DisplayVMOutputToStdout                   = false                               {product}
          bool DoEscapeAnalysis                          = true                                {C2 product}
          bool DontCompileHugeMethods                    = true                                {product}
          bool DontYieldALot                             = false                               {pd product}
         ccstr DumpLoadedClassList                       =                                     {product}
          bool DumpReplayDataOnError                     = true                                {product}
          bool DumpSharedSpaces                          = false                               {product}
          bool EagerXrunInit                             = false                               {product}
          intx EliminateAllocationArraySizeLimit         = 64                                  {C2 product}
          bool EliminateAllocations                      = true                                {C2 product}
          bool EliminateAutoBox                          = true                                {C2 product}
          bool EliminateLocks                            = true                                {C2 product}
          bool EliminateNestedLocks                      = true                                {C2 product}
          intx EmitSync                                  = 0                                   {product}
          bool EnableContended                           = true                                {product}
          bool EnableResourceManagementTLABCache         = true                                {product}
          bool EnableSharedLookupCache                   = true                                {product}
          bool EnableTracing                             = false                               {product}
         uintx ErgoHeapSizeLimit                         = 0                                   {product}
         ccstr ErrorFile                                 =                                     {product}
         ccstr ErrorReportServer                         =                                     {product}
        double EscapeAnalysisTimeout                     = 20.000000                           {C2 product}
          bool EstimateArgEscape                         = true                                {product}
          bool ExitOnOutOfMemoryError                    = false                               {product}
          bool ExplicitGCInvokesConcurrent               = false                               {product}
          bool ExplicitGCInvokesConcurrentAndUnloadsClasses  = false                               {product}
          bool ExtendedDTraceProbes                      = false                               {product}
         ccstr ExtraSharedClassListFile                  =                                     {product}
          bool FLSAlwaysCoalesceLarge                    = false                               {product}
         uintx FLSCoalescePolicy                         = 2                                   {product}
        double FLSLargestBlockCoalesceProximity          = 0.990000                            {product}
          bool FailOverToOldVerifier                     = true                                {product}
          bool FastTLABRefill                            = true                                {product}
          intx FenceInstruction                          = 0                                   {ARCH product}
          intx FieldsAllocationStyle                     = 1                                   {product}
          bool FilterSpuriousWakeups                     = true                                {product}
         ccstr FlightRecorderOptions                     =                                     {product}
          bool ForceNUMA                                 = false                               {product}
          bool ForceTimeHighResolution                   = false                               {product}
          intx FreqInlineSize                            = 325                                 {pd product}
        double G1ConcMarkStepDurationMillis              = 10.000000                           {product}
         uintx G1ConcRSHotCardLimit                      = 4                                   {product}
         uintx G1ConcRSLogCacheSize                      = 10                                  {product}
          intx G1ConcRefinementGreenZone                 = 0                                   {product}
          intx G1ConcRefinementRedZone                   = 0                                   {product}
          intx G1ConcRefinementServiceIntervalMillis     = 300                                 {product}
         uintx G1ConcRefinementThreads                   = 0                                   {product}
          intx G1ConcRefinementThresholdStep             = 0                                   {product}
          intx G1ConcRefinementYellowZone                = 0                                   {product}
         uintx G1ConfidencePercent                       = 50                                  {product}
         uintx G1HeapRegionSize                          = 0                                   {product}
         uintx G1HeapWastePercent                        = 5                                   {product}
         uintx G1MixedGCCountTarget                      = 8                                   {product}
          intx G1RSetRegionEntries                       = 0                                   {product}
         uintx G1RSetScanBlockSize                       = 64                                  {product}
          intx G1RSetSparseRegionEntries                 = 0                                   {product}
          intx G1RSetUpdatingPauseTimePercent            = 10                                  {product}
          intx G1RefProcDrainInterval                    = 10                                  {product}
         uintx G1ReservePercent                          = 10                                  {product}
         uintx G1SATBBufferEnqueueingThresholdPercent    = 60                                  {product}
          intx G1SATBBufferSize                          = 1024                                {product}
          intx G1UpdateBufferSize                        = 256                                 {product}
          bool G1UseAdaptiveConcRefinement               = true                                {product}
         uintx GCDrainStackTargetSize                    = 64                                  {product}
         uintx GCHeapFreeLimit                           = 2                                   {product}
         uintx GCLockerEdenExpansionPercent              = 5                                   {product}
          bool GCLockerInvokesConcurrent                 = false                               {product}
         uintx GCLogFileSize                             = 8192                                {product}
         uintx GCPauseIntervalMillis                     = 0                                   {product}
         uintx GCTaskTimeStampEntries                    = 200                                 {product}
         uintx GCTimeLimit                               = 98                                  {product}
         uintx GCTimeRatio                               = 99                                  {product}
         uintx HeapBaseMinAddress                        = 2147483648                          {pd product}
          bool HeapDumpAfterFullGC                       = false                               {manageable}
          bool HeapDumpBeforeFullGC                      = false                               {manageable}
          bool HeapDumpOnOutOfMemoryError                = false                               {manageable}
         ccstr HeapDumpPath                              =                                     {manageable}
         uintx HeapFirstMaximumCompactionCount           = 3                                   {product}
         uintx HeapMaximumCompactionInterval             = 20                                  {product}
         uintx HeapSizePerGCThread                       = 87241520                            {product}
          bool IgnoreEmptyClassPaths                     = false                               {product}
          bool IgnoreUnrecognizedVMOptions               = false                               {product}
         uintx IncreaseFirstTierCompileThresholdAt       = 50                                  {product}
          bool IncrementalInline                         = true                                {C2 product}
         uintx InitialBootClassLoaderMetaspaceSize       = 4194304                             {product}
         uintx InitialCodeCacheSize                      = 2555904                             {pd product}
         uintx InitialHeapSize                           = 0                                   {product}
         uintx InitialRAMFraction                        = 64                                  {product}
        double InitialRAMPercentage                      = 1.562500                            {product}
         uintx InitialSurvivorRatio                      = 8                                   {product}
         uintx InitialTenuringThreshold                  = 7                                   {product}
         uintx InitiatingHeapOccupancyPercent            = 45                                  {product}
          bool Inline                                    = true                                {product}
         ccstr InlineDataFile                            =                                     {product}
          intx InlineSmallCode                           = 1000                                {pd product}
          bool InlineSynchronizedMethods                 = true                                {C1 product}
          bool InsertMemBarAfterArraycopy                = true                                {C2 product}
          intx InteriorEntryAlignment                    = 16                                  {C2 pd product}
          intx InterpreterProfilePercentage              = 33                                  {product}
          bool JNIDetachReleasesMonitors                 = true                                {product}
          bool JavaMonitorsInStackTrace                  = true                                {product}
          intx JavaPriority10_To_OSPriority              = -1                                  {product}
          intx JavaPriority1_To_OSPriority               = -1                                  {product}
          intx JavaPriority2_To_OSPriority               = -1                                  {product}
          intx JavaPriority3_To_OSPriority               = -1                                  {product}
          intx JavaPriority4_To_OSPriority               = -1                                  {product}
          intx JavaPriority5_To_OSPriority               = -1                                  {product}
          intx JavaPriority6_To_OSPriority               = -1                                  {product}
          intx JavaPriority7_To_OSPriority               = -1                                  {product}
          intx JavaPriority8_To_OSPriority               = -1                                  {product}
          intx JavaPriority9_To_OSPriority               = -1                                  {product}
          bool LIRFillDelaySlots                         = false                               {C1 pd product}
         uintx LargePageHeapSizeThreshold                = 134217728                           {product}
         uintx LargePageSizeInBytes                      = 0                                   {product}
          bool LazyBootClassLoader                       = true                                {product}
          intx LiveNodeCountInliningCutoff               = 40000                               {C2 product}
          bool LogCommercialFeatures                     = false                               {product}
          intx LoopMaxUnroll                             = 16                                  {C2 product}
          intx LoopOptsCount                             = 43                                  {C2 product}
          intx LoopUnrollLimit                           = 60                                  {C2 pd product}
          intx LoopUnrollMin                             = 4                                   {C2 product}
          bool LoopUnswitching                           = true                                {C2 product}
          bool ManagementServer                          = false                               {product}
         uintx MarkStackSize                             = 4194304                             {product}
         uintx MarkStackSizeMax                          = 536870912                           {product}
         uintx MarkSweepAlwaysCompactCount               = 4                                   {product}
         uintx MarkSweepDeadRatio                        = 5                                   {product}
          intx MaxBCEAEstimateLevel                      = 5                                   {product}
          intx MaxBCEAEstimateSize                       = 150                                 {product}
         uintx MaxDirectMemorySize                       = 0                                   {product}
          bool MaxFDLimit                                = true                                {product}
         uintx MaxGCMinorPauseMillis                     = 4294967295                          {product}
         uintx MaxGCPauseMillis                          = 4294967295                          {product}
         uintx MaxHeapFreeRatio                          = 70                                  {manageable}
         uintx MaxHeapSize                               = 130862280                           {product}
          intx MaxInlineLevel                            = 9                                   {product}
          intx MaxInlineSize                             = 35                                  {product}
          intx MaxJNILocalCapacity                       = 65536                               {product}
          intx MaxJavaStackTraceDepth                    = 1024                                {product}
          intx MaxJumpTableSize                          = 65000                               {C2 product}
          intx MaxJumpTableSparseness                    = 5                                   {C2 product}
          intx MaxLabelRootDepth                         = 1100                                {C2 product}
          intx MaxLoopPad                                = 15                                  {C2 product}
         uintx MaxMetaspaceExpansion                     = 5452592                             {product}
         uintx MaxMetaspaceFreeRatio                     = 70                                  {product}
         uintx MaxMetaspaceSize                          = 4294967295                          {product}
         uintx MaxNewSize                                = 4294967295                          {product}
          intx MaxNodeLimit                              = 80000                               {C2 product}
      uint64_t MaxRAM                                    = 0                                   {pd product}
         uintx MaxRAMFraction                            = 4                                   {product}
        double MaxRAMPercentage                          = 25.000000                           {product}
          intx MaxRecursiveInlineLevel                   = 1                                   {product}
         uintx MaxTenuringThreshold                      = 15                                  {product}
          intx MaxTrivialSize                            = 6                                   {product}
          intx MaxVectorSize                             = 32                                  {C2 product}
         uintx MetaspaceSize                             = 21810376                            {pd product}
          bool MethodFlushing                            = true                                {product}
         uintx MinHeapDeltaBytes                         = 170392                              {product}
         uintx MinHeapFreeRatio                          = 40                                  {manageable}
          intx MinInliningThreshold                      = 250                                 {product}
          intx MinJumpTableSize                          = 10                                  {C2 pd product}
         uintx MinMetaspaceExpansion                     = 340784                              {product}
         uintx MinMetaspaceFreeRatio                     = 40                                  {product}
         uintx MinRAMFraction                            = 2                                   {product}
        double MinRAMPercentage                          = 50.000000                           {product}
         uintx MinSurvivorRatio                          = 3                                   {product}
         uintx MinTLABSize                               = 2048                                {product}
          intx MonitorBound                              = 0                                   {product}
          bool MonitorInUseLists                         = false                               {product}
          intx MultiArrayExpandLimit                     = 6                                   {C2 product}
          bool MustCallLoadClassInternal                 = false                               {product}
         uintx NUMAChunkResizeWeight                     = 20                                  {product}
         uintx NUMAInterleaveGranularity                 = 2097152                             {product}
         uintx NUMAPageScanRate                          = 256                                 {product}
         uintx NUMASpaceResizeRate                       = 1073741824                          {product}
          bool NUMAStats                                 = false                               {product}
         ccstr NativeMemoryTracking                      = off                                 {product}
          bool NeedsDeoptSuspend                         = false                               {pd product}
          bool NeverActAsServerClassMachine              = false                               {pd product}
          bool NeverTenure                               = false                               {product}
         uintx NewRatio                                  = 2                                   {product}
         uintx NewSize                                   = 1363144                             {product}
         uintx NewSizeThreadIncrease                     = 5320                                {pd product}
          intx NmethodSweepActivity                      = 10                                  {product}
          intx NmethodSweepCheckInterval                 = 5                                   {product}
          intx NmethodSweepFraction                      = 16                                  {product}
          intx NodeLimitFudgeFactor                      = 2000                                {C2 product}
         uintx NumberOfGCLogFiles                        = 0                                   {product}
          intx NumberOfLoopInstrToAlign                  = 4                                   {C2 product}
          intx ObjectAlignmentInBytes                    = 8                                   {lp64_product}
         uintx OldPLABSize                               = 1024                                {product}
         uintx OldPLABWeight                             = 50                                  {product}
         uintx OldSize                                   = 5452592                             {product}
          bool OmitStackTraceInFastThrow                 = true                                {product}
     ccstrlist OnError                                   =                                     {product}
     ccstrlist OnOutOfMemoryError                        =                                     {product}
          intx OnStackReplacePercentage                  = 140                                 {pd product}
          bool OptimizeFill                              = true                                {C2 product}
          bool OptimizePtrCompare                        = true                                {C2 product}
          bool OptimizeStringConcat                      = true                                {C2 product}
          bool OptoBundling                              = false                               {C2 pd product}
          intx OptoLoopAlignment                         = 16                                  {pd product}
          bool OptoScheduling                            = false                               {C2 pd product}
         uintx PLABWeight                                = 75                                  {product}
          bool PSChunkLargeArrays                        = true                                {product}
          intx ParGCArrayScanChunk                       = 50                                  {product}
         uintx ParGCDesiredObjsFromOverflowList          = 20                                  {product}
          bool ParGCTrimOverflow                         = true                                {product}
          bool ParGCUseLocalOverflow                     = false                               {product}
         uintx ParallelGCBufferWastePct                  = 10                                  {product}
         uintx ParallelGCThreads                         = 0                                   {product}
          bool ParallelGCVerbose                         = false                               {product}
         uintx ParallelOldDeadWoodLimiterMean            = 50                                  {product}
         uintx ParallelOldDeadWoodLimiterStdDev          = 80                                  {product}
          bool ParallelRefProcBalancingEnabled           = true                                {product}
          bool ParallelRefProcEnabled                    = false                               {product}
          bool PartialPeelAtUnsignedTests                = true                                {C2 product}
          bool PartialPeelLoop                           = true                                {C2 product}
          intx PartialPeelNewPhiDelta                    = 0                                   {C2 product}
         uintx PausePadding                              = 1                                   {product}
          intx PerBytecodeRecompilationCutoff            = 200                                 {product}
          intx PerBytecodeTrapLimit                      = 4                                   {product}
          intx PerMethodRecompilationCutoff              = 400                                 {product}
          intx PerMethodTrapLimit                        = 100                                 {product}
          bool PerfAllowAtExitRegistration               = false                               {product}
          bool PerfBypassFileSystemCheck                 = false                               {product}
          intx PerfDataMemorySize                        = 32768                               {product}
          intx PerfDataSamplingInterval                  = 50                                  {product}
         ccstr PerfDataSaveFile                          =                                     {product}
          bool PerfDataSaveToFile                        = false                               {product}
          bool PerfDisableSharedMem                      = false                               {product}
          intx PerfMaxStringConstLength                  = 1024                                {product}
          intx PreInflateSpin                            = 10                                  {pd product}
          bool PreferInterpreterNativeStubs              = false                               {pd product}
          intx PrefetchCopyIntervalInBytes               = -1                                  {product}
          intx PrefetchFieldsAhead                       = -1                                  {product}
          intx PrefetchScanIntervalInBytes               = -1                                  {product}
          bool PreserveAllAnnotations                    = false                               {product}
          bool PreserveFramePointer                      = false                               {pd product}
         uintx PretenureSizeThreshold                    = 0                                   {product}
          bool PrintAdaptiveSizePolicy                   = false                               {product}
          bool PrintCMSInitiationStatistics              = false                               {product}
          intx PrintCMSStatistics                        = 0                                   {product}
          bool PrintClassHistogram                       = false                               {manageable}
          bool PrintClassHistogramAfterFullGC            = false                               {manageable}
          bool PrintClassHistogramBeforeFullGC           = false                               {manageable}
          bool PrintCodeCache                            = false                               {product}
          bool PrintCodeCacheOnCompilation               = false                               {product}
          bool PrintCommandLineFlags                     = false                               {product}
          bool PrintCompilation                          = false                               {product}
          bool PrintConcurrentLocks                      = false                               {manageable}
          intx PrintFLSCensus                            = 0                                   {product}
          intx PrintFLSStatistics                        = 0                                   {product}
          bool PrintFlagsFinal                           = false                               {product}
          bool PrintFlagsInitial                         = false                               {product}
          bool PrintGC                                   = false                               {manageable}
          bool PrintGCApplicationConcurrentTime          = false                               {product}
          bool PrintGCApplicationStoppedTime             = false                               {product}
          bool PrintGCCause                              = true                                {product}
          bool PrintGCDateStamps                         = false                               {manageable}
          bool PrintGCDetails                            = false                               {manageable}
          bool PrintGCID                                 = false                               {manageable}
          bool PrintGCTaskTimeStamps                     = false                               {product}
          bool PrintGCTimeStamps                         = false                               {manageable}
          bool PrintHeapAtGC                             = false                               {product rw}
          bool PrintHeapAtGCExtended                     = false                               {product rw}
          bool PrintHeapAtSIGBREAK                       = true                                {product}
          bool PrintJNIGCStalls                          = false                               {product}
          bool PrintJNIResolving                         = false                               {product}
          bool PrintOldPLAB                              = false                               {product}
          bool PrintOopAddress                           = false                               {product}
          bool PrintPLAB                                 = false                               {product}
          bool PrintParallelOldGCPhaseTimes              = false                               {product}
          bool PrintPromotionFailure                     = false                               {product}
          bool PrintReferenceGC                          = false                               {product}
          bool PrintSafepointStatistics                  = false                               {product}
          intx PrintSafepointStatisticsCount             = 300                                 {product}
          intx PrintSafepointStatisticsTimeout           = -1                                  {product}
          bool PrintSharedArchiveAndExit                 = false                               {product}
          bool PrintSharedDictionary                     = false                               {product}
          bool PrintSharedSpaces                         = false                               {product}
          bool PrintStringDeduplicationStatistics        = false                               {product}
          bool PrintStringTableStatistics                = false                               {product}
          bool PrintTLAB                                 = false                               {product}
          bool PrintTenuringDistribution                 = false                               {product}
          bool PrintTieredEvents                         = false                               {product}
          bool PrintVMOptions                            = false                               {product}
          bool PrintVMQWaitTime                          = false                               {product}
          bool PrintWarnings                             = true                                {product}
         uintx ProcessDistributionStride                 = 4                                   {product}
          bool ProfileInterpreter                        = true                                {pd product}
          bool ProfileIntervals                          = false                               {product}
          intx ProfileIntervalsTicks                     = 100                                 {product}
          intx ProfileMaturityPercentage                 = 20                                  {product}
          bool ProfileVM                                 = false                               {product}
          bool ProfilerPrintByteCodeStatistics           = false                               {product}
          bool ProfilerRecordPC                          = false                               {product}
         uintx PromotedPadding                           = 3                                   {product}
         uintx QueuedAllocationWarningCount              = 0                                   {product}
         uintx RTMRetryCount                             = 5                                   {ARCH product}
          bool RangeCheckElimination                     = true                                {product}
          intx ReadPrefetchInstr                         = 0                                   {ARCH product}
          bool ReassociateInvariants                     = true                                {C2 product}
          bool ReduceBulkZeroing                         = true                                {C2 product}
          bool ReduceFieldZeroing                        = true                                {C2 product}
          bool ReduceInitialCardMarks                    = true                                {C2 product}
          bool ReduceSignalUsage                         = false                               {product}
          intx RefDiscoveryPolicy                        = 0                                   {product}
          bool ReflectionWrapResolutionErrors            = true                                {product}
          bool RegisterFinalizersAtInit                  = true                                {product}
          bool RelaxAccessControlCheck                   = false                               {product}
         ccstr ReplayDataFile                            =                                     {product}
          bool RequireSharedSpaces                       = false                               {product}
         uintx ReservedCodeCacheSize                     = 50331648                            {pd product}
          bool ResizeOldPLAB                             = true                                {product}
          bool ResizePLAB                                = true                                {product}
          bool ResizeTLAB                                = true                                {pd product}
          bool RestoreMXCSROnJNICalls                    = false                               {product}
          bool RestrictContended                         = true                                {product}
          bool RewriteBytecodes                          = true                                {pd product}
          bool RewriteFrequentPairs                      = true                                {pd product}
          intx SafepointPollOffset                       = 256                                 {C1 pd product}
          intx SafepointSpinBeforeYield                  = 2000                                {product}
          bool SafepointTimeout                          = false                               {product}
          intx SafepointTimeoutDelay                     = 10000                               {product}
          bool ScavengeBeforeFullGC                      = true                                {product}
          intx SelfDestructTimer                         = 0                                   {product}
         uintx SharedBaseAddress                         = 0                                   {product}
         ccstr SharedClassListFile                       =                                     {product}
         uintx SharedMiscCodeSize                        = 122880                              {product}
         uintx SharedMiscDataSize                        = 4194304                             {product}
         uintx SharedReadOnlySize                        = 16777216                            {product}
         uintx SharedReadWriteSize                       = 16777216                            {product}
          bool ShowMessageBoxOnError                     = false                               {product}
          intx SoftRefLRUPolicyMSPerMB                   = 1000                                {product}
          bool SpecialEncodeISOArray                     = true                                {C2 product}
          bool SplitIfBlocks                             = true                                {C2 product}
          intx StackRedPages                             = 1                                   {pd product}
          intx StackShadowPages                          = 6                                   {pd product}
          bool StackTraceInThrowable                     = true                                {product}
          intx StackYellowPages                          = 3                                   {pd product}
          bool StartAttachListener                       = false                               {product}
          intx StarvationMonitorInterval                 = 200                                 {product}
          bool StressLdcRewrite                          = false                               {product}
         uintx StringDeduplicationAgeThreshold           = 3                                   {product}
         uintx StringTableSize                           = 60013                               {product}
          bool SuppressFatalErrorMessage                 = false                               {product}
         uintx SurvivorPadding                           = 3                                   {product}
         uintx SurvivorRatio                             = 8                                   {product}
          intx SuspendRetryCount                         = 50                                  {product}
          intx SuspendRetryDelay                         = 5                                   {product}
          intx SyncFlags                                 = 0                                   {product}
         ccstr SyncKnobs                                 =                                     {product}
          intx SyncVerbose                               = 0                                   {product}
         uintx TLABAllocationWeight                      = 35                                  {product}
         uintx TLABRefillWasteFraction                   = 64                                  {product}
         uintx TLABSize                                  = 0                                   {product}
          bool TLABStats                                 = true                                {product}
         uintx TLABWasteIncrement                        = 4                                   {product}
         uintx TLABWasteTargetPercent                    = 1                                   {product}
         uintx TargetPLABWastePct                        = 10                                  {product}
         uintx TargetSurvivorRatio                       = 50                                  {product}
         uintx TenuredGenerationSizeIncrement            = 20                                  {product}
         uintx TenuredGenerationSizeSupplement           = 80                                  {product}
         uintx TenuredGenerationSizeSupplementDecay      = 2                                   {product}
          intx ThreadPriorityPolicy                      = 0                                   {product}
          bool ThreadPriorityVerbose                     = false                               {product}
         uintx ThreadSafetyMargin                        = 52428800                            {product}
          intx ThreadStackSize                           = 0                                   {pd product}
         uintx ThresholdTolerance                        = 10                                  {product}
          intx Tier0BackedgeNotifyFreqLog                = 10                                  {product}
          intx Tier0InvokeNotifyFreqLog                  = 7                                   {product}
          intx Tier0ProfilingStartPercentage             = 200                                 {product}
          intx Tier23InlineeNotifyFreqLog                = 20                                  {product}
          intx Tier2BackEdgeThreshold                    = 0                                   {product}
          intx Tier2BackedgeNotifyFreqLog                = 14                                  {product}
          intx Tier2CompileThreshold                     = 0                                   {product}
          intx Tier2InvokeNotifyFreqLog                  = 11                                  {product}
          intx Tier3BackEdgeThreshold                    = 60000                               {product}
          intx Tier3BackedgeNotifyFreqLog                = 13                                  {product}
          intx Tier3CompileThreshold                     = 2000                                {product}
          intx Tier3DelayOff                             = 2                                   {product}
          intx Tier3DelayOn                              = 5                                   {product}
          intx Tier3InvocationThreshold                  = 200                                 {product}
          intx Tier3InvokeNotifyFreqLog                  = 10                                  {product}
          intx Tier3LoadFeedback                         = 5                                   {product}
          intx Tier3MinInvocationThreshold               = 100                                 {product}
          intx Tier4BackEdgeThreshold                    = 40000                               {product}
          intx Tier4CompileThreshold                     = 15000                               {product}
          intx Tier4InvocationThreshold                  = 5000                                {product}
          intx Tier4LoadFeedback                         = 3                                   {product}
          intx Tier4MinInvocationThreshold               = 600                                 {product}
          bool TieredCompilation                         = true                                {pd product}
          intx TieredCompileTaskTimeout                  = 50                                  {product}
          intx TieredRateUpdateMaxTime                   = 25                                  {product}
          intx TieredRateUpdateMinTime                   = 1                                   {product}
          intx TieredStopAtLevel                         = 4                                   {product}
          bool TimeLinearScan                            = false                               {C1 product}
          bool TraceBiasedLocking                        = false                               {product}
          bool TraceClassLoading                         = false                               {product rw}
          bool TraceClassLoadingPreorder                 = false                               {product}
          bool TraceClassPaths                           = false                               {product}
          bool TraceClassResolution                      = false                               {product}
          bool TraceClassUnloading                       = false                               {product rw}
          bool TraceDynamicGCThreads                     = false                               {product}
          bool TraceGen0Time                             = false                               {product}
          bool TraceGen1Time                             = false                               {product}
         ccstr TraceJVMTI                                =                                     {product}
          bool TraceLoaderConstraints                    = false                               {product rw}
          bool TraceMetadataHumongousAllocation          = false                               {product}
          bool TraceMonitorInflation                     = false                               {product}
          bool TraceParallelOldGCTasks                   = false                               {product}
          intx TraceRedefineClasses                      = 0                                   {product}
          bool TraceSafepointCleanupTime                 = false                               {product}
          bool TraceSharedLookupCache                    = false                               {product}
          bool TraceSuspendWaitFailures                  = false                               {product}
          intx TrackedInitializationLimit                = 50                                  {C2 product}
          bool TransmitErrorReport                       = false                               {product}
          bool TrapBasedNullChecks                       = false                               {pd product}
          bool TrapBasedRangeChecks                      = false                               {C2 pd product}
          intx TypeProfileArgsLimit                      = 2                                   {product}
         uintx TypeProfileLevel                          = 111                                 {pd product}
          intx TypeProfileMajorReceiverPercent           = 90                                  {C2 product}
          intx TypeProfileParmsLimit                     = 2                                   {product}
          intx TypeProfileWidth                          = 2                                   {product}
          intx UnguardOnExecutionViolation               = 0                                   {product}
          bool UnlinkSymbolsALot                         = false                               {product}
          bool Use486InstrsOnly                          = false                               {ARCH product}
          bool UseAES                                    = false                               {product}
          bool UseAESCTRIntrinsics                       = false                               {product}
          bool UseAESIntrinsics                          = false                               {product}
          intx UseAVX                                    = 99                                  {ARCH product}
          bool UseAdaptiveGCBoundary                     = false                               {product}
          bool UseAdaptiveGenerationSizePolicyAtMajorCollection  = true                                {product}
          bool UseAdaptiveGenerationSizePolicyAtMinorCollection  = true                                {product}
          bool UseAdaptiveNUMAChunkSizing                = true                                {product}
          bool UseAdaptiveSizeDecayMajorGCCost           = true                                {product}
          bool UseAdaptiveSizePolicy                     = true                                {product}
          bool UseAdaptiveSizePolicyFootprintGoal        = true                                {product}
          bool UseAdaptiveSizePolicyWithSystemGC         = false                               {product}
          bool UseAddressNop                             = false                               {ARCH product}
          bool UseAltSigs                                = false                               {product}
          bool UseAutoGCSelectPolicy                     = false                               {product}
          bool UseBMI1Instructions                       = false                               {ARCH product}
          bool UseBMI2Instructions                       = false                               {ARCH product}
          bool UseBiasedLocking                          = true                                {product}
          bool UseBimorphicInlining                      = true                                {C2 product}
          bool UseBoundThreads                           = true                                {product}
          bool UseCLMUL                                  = false                               {ARCH product}
          bool UseCMSBestFit                             = true                                {product}
          bool UseCMSCollectionPassing                   = true                                {product}
          bool UseCMSCompactAtFullCollection             = true                                {product}
          bool UseCMSInitiatingOccupancyOnly             = false                               {product}
          bool UseCRC32Intrinsics                        = false                               {product}
          bool UseCodeCacheFlushing                      = true                                {product}
          bool UseCompiler                               = true                                {product}
          bool UseCompilerSafepoints                     = true                                {product}
          bool UseCompressedClassPointers                = false                               {lp64_product}
          bool UseCompressedOops                         = false                               {lp64_product}
          bool UseConcMarkSweepGC                        = false                               {product}
          bool UseCondCardMark                           = false                               {C2 product}
          bool UseCountLeadingZerosInstruction           = false                               {ARCH product}
          bool UseCountTrailingZerosInstruction          = false                               {ARCH product}
          bool UseCountedLoopSafepoints                  = false                               {C2 product}
          bool UseCounterDecay                           = true                                {product}
          bool UseDivMod                                 = true                                {C2 product}
          bool UseDynamicNumberOfGCThreads               = false                               {product}
          bool UseFPUForSpilling                         = false                               {C2 product}
          bool UseFastAccessorMethods                    = true                                {product}
          bool UseFastEmptyMethods                       = true                                {product}
          bool UseFastJNIAccessors                       = true                                {product}
          bool UseFastStosb                              = false                               {ARCH product}
          bool UseG1GC                                   = false                               {product}
          bool UseGCLogFileRotation                      = false                               {product}
          bool UseGCOverheadLimit                        = true                                {product}
          bool UseGCTaskAffinity                         = false                               {product}
          bool UseGHASHIntrinsics                        = false                               {product}
          bool UseHeavyMonitors                          = false                               {product}
          bool UseInlineCaches                           = true                                {product}
          bool UseInterpreter                            = true                                {product}
          bool UseJumpTables                             = true                                {C2 product}
          bool UseLWPSynchronization                     = true                                {product}
          bool UseLargePages                             = false                               {pd product}
          bool UseLargePagesInMetaspace                  = false                               {product}
          bool UseLargePagesIndividualAllocation        := false                               {pd product}
          bool UseLockedTracing                          = false                               {product}
          bool UseLoopCounter                            = true                                {product}
          bool UseLoopInvariantCodeMotion                = true                                {C1 product}
          bool UseLoopPredicate                          = true                                {C2 product}
          bool UseMathExactIntrinsics                    = true                                {C2 product}
          bool UseMaximumCompactionOnSystemGC            = true                                {product}
          bool UseMembar                                 = false                               {pd product}
          bool UseMontgomeryMultiplyIntrinsic            = false                               {C2 product}
          bool UseMontgomerySquareIntrinsic              = false                               {C2 product}
          bool UseMulAddIntrinsic                        = false                               {C2 product}
          bool UseMultiplyToLenIntrinsic                 = false                               {C2 product}
          bool UseNUMA                                   = false                               {product}
          bool UseNUMAInterleaving                       = false                               {product}
          bool UseNewLongLShift                          = false                               {ARCH product}
          bool UseOSErrorReporting                       = false                               {pd product}
          bool UseOldInlining                            = true                                {C2 product}
          bool UseOnStackReplacement                     = true                                {pd product}
          bool UseOnlyInlinedBimorphic                   = true                                {C2 product}
          bool UseOptoBiasInlining                       = true                                {C2 product}
          bool UsePSAdaptiveSurvivorSizePolicy           = true                                {product}
          bool UseParNewGC                               = false                               {product}
          bool UseParallelGC                             = false                               {product}
          bool UseParallelOldGC                          = false                               {product}
          bool UsePerfData                               = true                                {product}
          bool UsePopCountInstruction                    = false                               {product}
          bool UseRDPCForConstantTableBase               = false                               {C2 product}
          bool UseRTMDeopt                               = false                               {ARCH product}
          bool UseRTMLocking                             = false                               {ARCH product}
          bool UseSHA                                    = false                               {product}
          bool UseSHA1Intrinsics                         = false                               {product}
          bool UseSHA256Intrinsics                       = false                               {product}
          bool UseSHA512Intrinsics                       = false                               {product}
          intx UseSSE                                    = 99                                  {product}
          bool UseSSE42Intrinsics                        = false                               {product}
          bool UseSerialGC                               = false                               {product}
          bool UseSharedSpaces                           = true                                {product}
          bool UseSignalChaining                         = true                                {product}
          bool UseSquareToLenIntrinsic                   = false                               {C2 product}
          bool UseStoreImmI16                            = true                                {ARCH product}
          bool UseStringDeduplication                    = false                               {product}
          bool UseSuperWord                              = true                                {C2 product}
          bool UseTLAB                                   = true                                {pd product}
          bool UseThreadPriorities                       = true                                {pd product}
          bool UseTypeProfile                            = true                                {product}
          bool UseTypeSpeculation                        = true                                {C2 product}
          bool UseUTCFileTimestamp                       = true                                {product}
          bool UseUnalignedLoadStores                    = false                               {ARCH product}
          bool UseVMInterruptibleIO                      = false                               {product}
          bool UseXMMForArrayCopy                        = false                               {product}
          bool UseXmmI2D                                 = false                               {ARCH product}
          bool UseXmmI2F                                 = false                               {ARCH product}
          bool UseXmmLoadAndClearUpper                   = true                                {ARCH product}
          bool UseXmmRegToRegMoveAll                     = false                               {ARCH product}
          bool VMThreadHintNoPreempt                     = false                               {product}
          intx VMThreadPriority                          = -1                                  {product}
          intx VMThreadStackSize                         = 0                                   {pd product}
          intx ValueMapInitialSize                       = 11                                  {C1 product}
          intx ValueMapMaxLoopSize                       = 8                                   {C1 product}
          intx ValueSearchLimit                          = 1000                                {C2 product}
          bool VerifyMergedCPBytecodes                   = true                                {product}
          bool VerifySharedSpaces                        = false                               {product}
          intx WorkAroundNPTLTimedWaitHang               = 1                                   {product}
         uintx YoungGenerationSizeIncrement              = 20                                  {product}
         uintx YoungGenerationSizeSupplement             = 80                                  {product}
         uintx YoungGenerationSizeSupplementDecay        = 8                                   {product}
         uintx YoungPLABSize                             = 4096                                {product}
          bool ZeroTLAB                                  = false                               {product}
          intx hashCode                                  = 5                                   {product}
     
     D:\IdeaProjects\my01>
     
     
     ```
     
     - 查看运行时生效的XX参数
     
     ```console
     # 查看XX参数运行程序时生效的值
     D:\IdeaProjects\my01>java -XX:+PrintFlagsFinal -version
     [Global flags]
          intx ActiveProcessorCount                      = -1                                  {product}
         uintx AdaptiveSizeDecrementScaleFactor          = 4                                   {product}
         uintx AdaptiveSizeMajorGCDecayTimeScale         = 10                                  {product}
         uintx AdaptiveSizePausePolicy                   = 0                                   {product}
         uintx AdaptiveSizePolicyCollectionCostMargin    = 50                                  {product}
         uintx AdaptiveSizePolicyInitializingSteps       = 20                                  {product}
         uintx AdaptiveSizePolicyOutputInterval          = 0                                   {product}
         uintx AdaptiveSizePolicyWeight                  = 10                                  {product}
         uintx AdaptiveSizeThroughPutPolicy              = 0                                   {product}
         uintx AdaptiveTimeWeight                        = 25                                  {product}
          bool AdjustConcurrency                         = false                               {product}
          bool AggressiveHeap                            = false                               {product}
          bool AggressiveOpts                            = false                               {product}
          intx AliasLevel                                = 3                                   {C2 product}
          bool AlignVector                               = false                               {C2 product}
          intx AllocateInstancePrefetchLines             = 1                                   {product}
          intx AllocatePrefetchDistance                  = 192                                 {product}
          intx AllocatePrefetchInstr                     = 3                                   {product}
          intx AllocatePrefetchLines                     = 4                                   {product}
          intx AllocatePrefetchStepSize                  = 64                                  {product}
          intx AllocatePrefetchStyle                     = 1                                   {product}
          bool AllowJNIEnvProxy                          = false                               {product}
          bool AllowNonVirtualCalls                      = false                               {product}
          bool AllowParallelDefineClass                  = false                               {product}
          bool AllowUserSignalHandlers                   = false                               {product}
          bool AlwaysActAsServerClassMachine             = false                               {product}
          bool AlwaysCompileLoopMethods                  = false                               {product}
          bool AlwaysLockClassLoader                     = false                               {product}
          bool AlwaysPreTouch                            = false                               {product}
          bool AlwaysRestoreFPU                          = false                               {product}
          bool AlwaysTenure                              = false                               {product}
          bool AssertOnSuspendWaitFailure                = false                               {product}
          bool AssumeMP                                  = false                               {product}
          intx AutoBoxCacheMax                           = 128                                 {C2 product}
         uintx AutoGCSelectPauseMillis                   = 5000                                {product}
          intx BCEATraceLevel                            = 0                                   {product}
          intx BackEdgeThreshold                         = 100000                              {pd product}
          bool BackgroundCompilation                     = true                                {pd product}
         uintx BaseFootPrintEstimate                     = 268435456                           {product}
          intx BiasedLockingBulkRebiasThreshold          = 20                                  {product}
          intx BiasedLockingBulkRevokeThreshold          = 40                                  {product}
          intx BiasedLockingDecayTime                    = 25000                               {product}
          intx BiasedLockingStartupDelay                 = 4000                                {product}
          bool BindGCTaskThreadsToCPUs                   = false                               {product}
          bool BlockLayoutByFrequency                    = true                                {C2 product}
          intx BlockLayoutMinDiamondPercentage           = 20                                  {C2 product}
          bool BlockLayoutRotateLoops                    = true                                {C2 product}
          bool BranchOnRegister                          = false                               {C2 product}
          bool BytecodeVerificationLocal                 = false                               {product}
          bool BytecodeVerificationRemote                = true                                {product}
          bool C1OptimizeVirtualCallProfiling            = true                                {C1 product}
          bool C1ProfileBranches                         = true                                {C1 product}
          bool C1ProfileCalls                            = true                                {C1 product}
          bool C1ProfileCheckcasts                       = true                                {C1 product}
          bool C1ProfileInlinedCalls                     = true                                {C1 product}
          bool C1ProfileVirtualCalls                     = true                                {C1 product}
          bool C1UpdateMethodData                        = true                                {C1 product}
          intx CICompilerCount                          := 4                                   {product}
          bool CICompilerCountPerCPU                     = true                                {product}
          bool CITime                                    = false                               {product}
          bool CMSAbortSemantics                         = false                               {product}
         uintx CMSAbortablePrecleanMinWorkPerIteration   = 100                                 {product}
          intx CMSAbortablePrecleanWaitMillis            = 100                                 {manageable}
         uintx CMSBitMapYieldQuantum                     = 10485760                            {product}
         uintx CMSBootstrapOccupancy                     = 50                                  {product}
          bool CMSClassUnloadingEnabled                  = true                                {product}
         uintx CMSClassUnloadingMaxInterval              = 0                                   {product}
          bool CMSCleanOnEnter                           = true                                {product}
          bool CMSCompactWhenClearAllSoftRefs            = true                                {product}
         uintx CMSConcMarkMultiple                       = 32                                  {product}
          bool CMSConcurrentMTEnabled                    = true                                {product}
         uintx CMSCoordinatorYieldSleepCount             = 10                                  {product}
          bool CMSDumpAtPromotionFailure                 = false                               {product}
          bool CMSEdenChunksRecordAlways                 = true                                {product}
         uintx CMSExpAvgFactor                           = 50                                  {product}
          bool CMSExtrapolateSweep                       = false                               {product}
         uintx CMSFullGCsBeforeCompaction                = 0                                   {product}
         uintx CMSIncrementalDutyCycle                   = 10                                  {product}
         uintx CMSIncrementalDutyCycleMin                = 0                                   {product}
          bool CMSIncrementalMode                        = false                               {product}
         uintx CMSIncrementalOffset                      = 0                                   {product}
          bool CMSIncrementalPacing                      = true                                {product}
         uintx CMSIncrementalSafetyFactor                = 10                                  {product}
         uintx CMSIndexedFreeListReplenish               = 4                                   {product}
          intx CMSInitiatingOccupancyFraction            = -1                                  {product}
         uintx CMSIsTooFullPercentage                    = 98                                  {product}
        double CMSLargeCoalSurplusPercent                = 0.950000                            {product}
        double CMSLargeSplitSurplusPercent               = 1.000000                            {product}
          bool CMSLoopWarn                               = false                               {product}
         uintx CMSMaxAbortablePrecleanLoops              = 0                                   {product}
          intx CMSMaxAbortablePrecleanTime               = 5000                                {product}
         uintx CMSOldPLABMax                             = 1024                                {product}
         uintx CMSOldPLABMin                             = 16                                  {product}
         uintx CMSOldPLABNumRefills                      = 4                                   {product}
         uintx CMSOldPLABReactivityFactor                = 2                                   {product}
          bool CMSOldPLABResizeQuicker                   = false                               {product}
         uintx CMSOldPLABToleranceFactor                 = 4                                   {product}
          bool CMSPLABRecordAlways                       = true                                {product}
         uintx CMSParPromoteBlocksToClaim                = 16                                  {product}
          bool CMSParallelInitialMarkEnabled             = true                                {product}
          bool CMSParallelRemarkEnabled                  = true                                {product}
          bool CMSParallelSurvivorRemarkEnabled          = true                                {product}
         uintx CMSPrecleanDenominator                    = 3                                   {product}
         uintx CMSPrecleanIter                           = 3                                   {product}
         uintx CMSPrecleanNumerator                      = 2                                   {product}
          bool CMSPrecleanRefLists1                      = true                                {product}
          bool CMSPrecleanRefLists2                      = false                               {product}
          bool CMSPrecleanSurvivors1                     = false                               {product}
          bool CMSPrecleanSurvivors2                     = true                                {product}
         uintx CMSPrecleanThreshold                      = 1000                                {product}
          bool CMSPrecleaningEnabled                     = true                                {product}
          bool CMSPrintChunksInDump                      = false                               {product}
          bool CMSPrintEdenSurvivorChunks                = false                               {product}
          bool CMSPrintObjectsInDump                     = false                               {product}
         uintx CMSRemarkVerifyVariant                    = 1                                   {product}
          bool CMSReplenishIntermediate                  = true                                {product}
         uintx CMSRescanMultiple                         = 32                                  {product}
         uintx CMSSamplingGrain                          = 16384                               {product}
          bool CMSScavengeBeforeRemark                   = false                               {product}
         uintx CMSScheduleRemarkEdenPenetration          = 50                                  {product}
         uintx CMSScheduleRemarkEdenSizeThreshold        = 2097152                             {product}
         uintx CMSScheduleRemarkSamplingRatio            = 5                                   {product}
        double CMSSmallCoalSurplusPercent                = 1.050000                            {product}
        double CMSSmallSplitSurplusPercent               = 1.100000                            {product}
          bool CMSSplitIndexedFreeListBlocks             = true                                {product}
          intx CMSTriggerInterval                        = -1                                  {manageable}
         uintx CMSTriggerRatio                           = 80                                  {product}
          intx CMSWaitDuration                           = 2000                                {manageable}
         uintx CMSWorkQueueDrainThreshold                = 10                                  {product}
          bool CMSYield                                  = true                                {product}
         uintx CMSYieldSleepCount                        = 0                                   {product}
         uintx CMSYoungGenPerWorker                      = 67108864                            {pd product}
         uintx CMS_FLSPadding                            = 1                                   {product}
         uintx CMS_FLSWeight                             = 75                                  {product}
         uintx CMS_SweepPadding                          = 1                                   {product}
         uintx CMS_SweepTimerThresholdMillis             = 10                                  {product}
         uintx CMS_SweepWeight                           = 75                                  {product}
          bool CheckEndorsedAndExtDirs                   = false                               {product}
          bool CheckJNICalls                             = false                               {product}
          bool ClassUnloading                            = true                                {product}
          bool ClassUnloadingWithConcurrentMark          = true                                {product}
          intx ClearFPUAtPark                            = 0                                   {product}
          bool ClipInlining                              = true                                {product}
         uintx CodeCacheExpansionSize                    = 65536                               {pd product}
         uintx CodeCacheMinimumFreeSpace                 = 512000                              {product}
          bool CollectGen0First                          = false                               {product}
          bool CompactFields                             = true                                {product}
          intx CompilationPolicyChoice                   = 3                                   {product}
     ccstrlist CompileCommand                            =                                     {product}
         ccstr CompileCommandFile                        =                                     {product}
     ccstrlist CompileOnly                               =                                     {product}
          intx CompileThreshold                          = 10000                               {pd product}
          bool CompilerThreadHintNoPreempt               = true                                {product}
          intx CompilerThreadPriority                    = -1                                  {product}
          intx CompilerThreadStackSize                   = 0                                   {pd product}
         uintx CompressedClassSpaceSize                  = 1073741824                          {product}
         uintx ConcGCThreads                             = 0                                   {product}
          intx ConditionalMoveLimit                      = 3                                   {C2 pd product}
          intx ContendedPaddingWidth                     = 128                                 {product}
          bool ConvertSleepToYield                       = true                                {pd product}
          bool ConvertYieldToSleep                       = false                               {product}
          bool CrashOnOutOfMemoryError                   = false                               {product}
          bool CreateMinidumpOnCrash                     = false                               {product}
          bool CriticalJNINatives                        = true                                {product}
          bool DTraceAllocProbes                         = false                               {product}
          bool DTraceMethodProbes                        = false                               {product}
          bool DTraceMonitorProbes                       = false                               {product}
          bool Debugging                                 = false                               {product}
         uintx DefaultMaxRAMFraction                     = 4                                   {product}
          intx DefaultThreadPriority                     = -1                                  {product}
          intx DeferPollingPageLoopCount                 = -1                                  {product}
          intx DeferThrSuspendLoopCount                  = 4000                                {product}
          bool DeoptimizeRandom                          = false                               {product}
          bool DisableAttachMechanism                    = false                               {product}
          bool DisableExplicitGC                         = false                               {product}
          bool DisplayVMOutputToStderr                   = false                               {product}
          bool DisplayVMOutputToStdout                   = false                               {product}
          bool DoEscapeAnalysis                          = true                                {C2 product}
          bool DontCompileHugeMethods                    = true                                {product}
          bool DontYieldALot                             = false                               {pd product}
         ccstr DumpLoadedClassList                       =                                     {product}
          bool DumpReplayDataOnError                     = true                                {product}
          bool DumpSharedSpaces                          = false                               {product}
          bool EagerXrunInit                             = false                               {product}
          intx EliminateAllocationArraySizeLimit         = 64                                  {C2 product}
          bool EliminateAllocations                      = true                                {C2 product}
          bool EliminateAutoBox                          = true                                {C2 product}
          bool EliminateLocks                            = true                                {C2 product}
          bool EliminateNestedLocks                      = true                                {C2 product}
          intx EmitSync                                  = 0                                   {product}
          bool EnableContended                           = true                                {product}
          bool EnableResourceManagementTLABCache         = true                                {product}
          bool EnableSharedLookupCache                   = true                                {product}
          bool EnableTracing                             = false                               {product}
         uintx ErgoHeapSizeLimit                         = 0                                   {product}
         ccstr ErrorFile                                 =                                     {product}
         ccstr ErrorReportServer                         =                                     {product}
        double EscapeAnalysisTimeout                     = 20.000000                           {C2 product}
          bool EstimateArgEscape                         = true                                {product}
          bool ExitOnOutOfMemoryError                    = false                               {product}
          bool ExplicitGCInvokesConcurrent               = false                               {product}
          bool ExplicitGCInvokesConcurrentAndUnloadsClasses  = false                               {product}
          bool ExtendedDTraceProbes                      = false                               {product}
         ccstr ExtraSharedClassListFile                  =                                     {product}
          bool FLSAlwaysCoalesceLarge                    = false                               {product}
         uintx FLSCoalescePolicy                         = 2                                   {product}
        double FLSLargestBlockCoalesceProximity          = 0.990000                            {product}
          bool FailOverToOldVerifier                     = true                                {product}
          bool FastTLABRefill                            = true                                {product}
          intx FenceInstruction                          = 0                                   {ARCH product}
          intx FieldsAllocationStyle                     = 1                                   {product}
          bool FilterSpuriousWakeups                     = true                                {product}
         ccstr FlightRecorderOptions                     =                                     {product}
          bool ForceNUMA                                 = false                               {product}
          bool ForceTimeHighResolution                   = false                               {product}
          intx FreqInlineSize                            = 325                                 {pd product}
        double G1ConcMarkStepDurationMillis              = 10.000000                           {product}
         uintx G1ConcRSHotCardLimit                      = 4                                   {product}
         uintx G1ConcRSLogCacheSize                      = 10                                  {product}
          intx G1ConcRefinementGreenZone                 = 0                                   {product}
          intx G1ConcRefinementRedZone                   = 0                                   {product}
          intx G1ConcRefinementServiceIntervalMillis     = 300                                 {product}
         uintx G1ConcRefinementThreads                   = 0                                   {product}
          intx G1ConcRefinementThresholdStep             = 0                                   {product}
          intx G1ConcRefinementYellowZone                = 0                                   {product}
         uintx G1ConfidencePercent                       = 50                                  {product}
         uintx G1HeapRegionSize                          = 0                                   {product}
         uintx G1HeapWastePercent                        = 5                                   {product}
         uintx G1MixedGCCountTarget                      = 8                                   {product}
          intx G1RSetRegionEntries                       = 0                                   {product}
         uintx G1RSetScanBlockSize                       = 64                                  {product}
          intx G1RSetSparseRegionEntries                 = 0                                   {product}
          intx G1RSetUpdatingPauseTimePercent            = 10                                  {product}
          intx G1RefProcDrainInterval                    = 10                                  {product}
         uintx G1ReservePercent                          = 10                                  {product}
         uintx G1SATBBufferEnqueueingThresholdPercent    = 60                                  {product}
          intx G1SATBBufferSize                          = 1024                                {product}
          intx G1UpdateBufferSize                        = 256                                 {product}
          bool G1UseAdaptiveConcRefinement               = true                                {product}
         uintx GCDrainStackTargetSize                    = 64                                  {product}
         uintx GCHeapFreeLimit                           = 2                                   {product}
         uintx GCLockerEdenExpansionPercent              = 5                                   {product}
          bool GCLockerInvokesConcurrent                 = false                               {product}
         uintx GCLogFileSize                             = 8192                                {product}
         uintx GCPauseIntervalMillis                     = 0                                   {product}
         uintx GCTaskTimeStampEntries                    = 200                                 {product}
         uintx GCTimeLimit                               = 98                                  {product}
         uintx GCTimeRatio                               = 99                                  {product}
         uintx HeapBaseMinAddress                        = 2147483648                          {pd product}
          bool HeapDumpAfterFullGC                       = false                               {manageable}
          bool HeapDumpBeforeFullGC                      = false                               {manageable}
          bool HeapDumpOnOutOfMemoryError                = false                               {manageable}
         ccstr HeapDumpPath                              =                                     {manageable}
         uintx HeapFirstMaximumCompactionCount           = 3                                   {product}
         uintx HeapMaximumCompactionInterval             = 20                                  {product}
         uintx HeapSizePerGCThread                       = 87241520                            {product}
          bool IgnoreEmptyClassPaths                     = false                               {product}
          bool IgnoreUnrecognizedVMOptions               = false                               {product}
         uintx IncreaseFirstTierCompileThresholdAt       = 50                                  {product}
          bool IncrementalInline                         = true                                {C2 product}
         uintx InitialBootClassLoaderMetaspaceSize       = 4194304                             {product}
         uintx InitialCodeCacheSize                      = 2555904                             {pd product}
         uintx InitialHeapSize                          := 260046848                           {product}
         uintx InitialRAMFraction                        = 64                                  {product}
        double InitialRAMPercentage                      = 1.562500                            {product}
         uintx InitialSurvivorRatio                      = 8                                   {product}
         uintx InitialTenuringThreshold                  = 7                                   {product}
         uintx InitiatingHeapOccupancyPercent            = 45                                  {product}
          bool Inline                                    = true                                {product}
         ccstr InlineDataFile                            =                                     {product}
          intx InlineSmallCode                           = 2000                                {pd product}
          bool InlineSynchronizedMethods                 = true                                {C1 product}
          bool InsertMemBarAfterArraycopy                = true                                {C2 product}
          intx InteriorEntryAlignment                    = 16                                  {C2 pd product}
          intx InterpreterProfilePercentage              = 33                                  {product}
          bool JNIDetachReleasesMonitors                 = true                                {product}
          bool JavaMonitorsInStackTrace                  = true                                {product}
          intx JavaPriority10_To_OSPriority              = -1                                  {product}
          intx JavaPriority1_To_OSPriority               = -1                                  {product}
          intx JavaPriority2_To_OSPriority               = -1                                  {product}
          intx JavaPriority3_To_OSPriority               = -1                                  {product}
          intx JavaPriority4_To_OSPriority               = -1                                  {product}
          intx JavaPriority5_To_OSPriority               = -1                                  {product}
          intx JavaPriority6_To_OSPriority               = -1                                  {product}
          intx JavaPriority7_To_OSPriority               = -1                                  {product}
          intx JavaPriority8_To_OSPriority               = -1                                  {product}
          intx JavaPriority9_To_OSPriority               = -1                                  {product}
          bool LIRFillDelaySlots                         = false                               {C1 pd product}
         uintx LargePageHeapSizeThreshold                = 134217728                           {product}
         uintx LargePageSizeInBytes                      = 0                                   {product}
          bool LazyBootClassLoader                       = true                                {product}
          intx LiveNodeCountInliningCutoff               = 40000                               {C2 product}
          bool LogCommercialFeatures                     = false                               {product}
          intx LoopMaxUnroll                             = 16                                  {C2 product}
          intx LoopOptsCount                             = 43                                  {C2 product}
          intx LoopUnrollLimit                           = 60                                  {C2 pd product}
          intx LoopUnrollMin                             = 4                                   {C2 product}
          bool LoopUnswitching                           = true                                {C2 product}
          bool ManagementServer                          = false                               {product}
         uintx MarkStackSize                             = 4194304                             {product}
         uintx MarkStackSizeMax                          = 536870912                           {product}
         uintx MarkSweepAlwaysCompactCount               = 4                                   {product}
         uintx MarkSweepDeadRatio                        = 1                                   {product}
          intx MaxBCEAEstimateLevel                      = 5                                   {product}
          intx MaxBCEAEstimateSize                       = 150                                 {product}
         uintx MaxDirectMemorySize                       = 0                                   {product}
          bool MaxFDLimit                                = true                                {product}
         uintx MaxGCMinorPauseMillis                     = 4294967295                          {product}
         uintx MaxGCPauseMillis                          = 4294967295                          {product}
         uintx MaxHeapFreeRatio                          = 100                                 {manageable}
         uintx MaxHeapSize                              := 4139778048                          {product}
          intx MaxInlineLevel                            = 9                                   {product}
          intx MaxInlineSize                             = 35                                  {product}
          intx MaxJNILocalCapacity                       = 65536                               {product}
          intx MaxJavaStackTraceDepth                    = 1024                                {product}
          intx MaxJumpTableSize                          = 65000                               {C2 product}
          intx MaxJumpTableSparseness                    = 5                                   {C2 product}
          intx MaxLabelRootDepth                         = 1100                                {C2 product}
          intx MaxLoopPad                                = 11                                  {C2 product}
         uintx MaxMetaspaceExpansion                     = 5451776                             {product}
         uintx MaxMetaspaceFreeRatio                     = 70                                  {product}
         uintx MaxMetaspaceSize                          = 4294901760                          {product}
         uintx MaxNewSize                               := 1379926016                          {product}
          intx MaxNodeLimit                              = 75000                               {C2 product}
      uint64_t MaxRAM                                    = 0                                   {pd product}
         uintx MaxRAMFraction                            = 4                                   {product}
        double MaxRAMPercentage                          = 25.000000                           {product}
          intx MaxRecursiveInlineLevel                   = 1                                   {product}
         uintx MaxTenuringThreshold                      = 15                                  {product}
          intx MaxTrivialSize                            = 6                                   {product}
          intx MaxVectorSize                             = 32                                  {C2 product}
         uintx MetaspaceSize                             = 21807104                            {pd product}
          bool MethodFlushing                            = true                                {product}
         uintx MinHeapDeltaBytes                        := 524288                              {product}
         uintx MinHeapFreeRatio                          = 0                                   {manageable}
          intx MinInliningThreshold                      = 250                                 {product}
          intx MinJumpTableSize                          = 10                                  {C2 pd product}
         uintx MinMetaspaceExpansion                     = 339968                              {product}
         uintx MinMetaspaceFreeRatio                     = 40                                  {product}
         uintx MinRAMFraction                            = 2                                   {product}
        double MinRAMPercentage                          = 50.000000                           {product}
         uintx MinSurvivorRatio                          = 3                                   {product}
         uintx MinTLABSize                               = 2048                                {product}
          intx MonitorBound                              = 0                                   {product}
          bool MonitorInUseLists                         = false                               {product}
          intx MultiArrayExpandLimit                     = 6                                   {C2 product}
          bool MustCallLoadClassInternal                 = false                               {product}
         uintx NUMAChunkResizeWeight                     = 20                                  {product}
         uintx NUMAInterleaveGranularity                 = 2097152                             {product}
         uintx NUMAPageScanRate                          = 256                                 {product}
         uintx NUMASpaceResizeRate                       = 1073741824                          {product}
          bool NUMAStats                                 = false                               {product}
         ccstr NativeMemoryTracking                      = off                                 {product}
          bool NeedsDeoptSuspend                         = false                               {pd product}
          bool NeverActAsServerClassMachine              = false                               {pd product}
          bool NeverTenure                               = false                               {product}
         uintx NewRatio                                  = 2                                   {product}
         uintx NewSize                                  := 86507520                            {product}
         uintx NewSizeThreadIncrease                     = 5320                                {pd product}
          intx NmethodSweepActivity                      = 10                                  {product}
          intx NmethodSweepCheckInterval                 = 5                                   {product}
          intx NmethodSweepFraction                      = 16                                  {product}
          intx NodeLimitFudgeFactor                      = 2000                                {C2 product}
         uintx NumberOfGCLogFiles                        = 0                                   {product}
          intx NumberOfLoopInstrToAlign                  = 4                                   {C2 product}
          intx ObjectAlignmentInBytes                    = 8                                   {lp64_product}
         uintx OldPLABSize                               = 1024                                {product}
         uintx OldPLABWeight                             = 50                                  {product}
         uintx OldSize                                  := 173539328                           {product}
          bool OmitStackTraceInFastThrow                 = true                                {product}
     ccstrlist OnError                                   =                                     {product}
     ccstrlist OnOutOfMemoryError                        =                                     {product}
          intx OnStackReplacePercentage                  = 140                                 {pd product}
          bool OptimizeFill                              = true                                {C2 product}
          bool OptimizePtrCompare                        = true                                {C2 product}
          bool OptimizeStringConcat                      = true                                {C2 product}
          bool OptoBundling                              = false                               {C2 pd product}
          intx OptoLoopAlignment                         = 16                                  {pd product}
          bool OptoScheduling                            = false                               {C2 pd product}
         uintx PLABWeight                                = 75                                  {product}
          bool PSChunkLargeArrays                        = true                                {product}
          intx ParGCArrayScanChunk                       = 50                                  {product}
         uintx ParGCDesiredObjsFromOverflowList          = 20                                  {product}
          bool ParGCTrimOverflow                         = true                                {product}
          bool ParGCUseLocalOverflow                     = false                               {product}
         uintx ParallelGCBufferWastePct                  = 10                                  {product}
         uintx ParallelGCThreads                         = 8                                   {product}
          bool ParallelGCVerbose                         = false                               {product}
         uintx ParallelOldDeadWoodLimiterMean            = 50                                  {product}
         uintx ParallelOldDeadWoodLimiterStdDev          = 80                                  {product}
          bool ParallelRefProcBalancingEnabled           = true                                {product}
          bool ParallelRefProcEnabled                    = false                               {product}
          bool PartialPeelAtUnsignedTests                = true                                {C2 product}
          bool PartialPeelLoop                           = true                                {C2 product}
          intx PartialPeelNewPhiDelta                    = 0                                   {C2 product}
         uintx PausePadding                              = 1                                   {product}
          intx PerBytecodeRecompilationCutoff            = 200                                 {product}
          intx PerBytecodeTrapLimit                      = 4                                   {product}
          intx PerMethodRecompilationCutoff              = 400                                 {product}
          intx PerMethodTrapLimit                        = 100                                 {product}
          bool PerfAllowAtExitRegistration               = false                               {product}
          bool PerfBypassFileSystemCheck                 = false                               {product}
          intx PerfDataMemorySize                        = 32768                               {product}
          intx PerfDataSamplingInterval                  = 50                                  {product}
         ccstr PerfDataSaveFile                          =                                     {product}
          bool PerfDataSaveToFile                        = false                               {product}
          bool PerfDisableSharedMem                      = false                               {product}
          intx PerfMaxStringConstLength                  = 1024                                {product}
          intx PreInflateSpin                            = 10                                  {pd product}
          bool PreferInterpreterNativeStubs              = false                               {pd product}
          intx PrefetchCopyIntervalInBytes               = 576                                 {product}
          intx PrefetchFieldsAhead                       = 1                                   {product}
          intx PrefetchScanIntervalInBytes               = 576                                 {product}
          bool PreserveAllAnnotations                    = false                               {product}
          bool PreserveFramePointer                      = false                               {pd product}
         uintx PretenureSizeThreshold                    = 0                                   {product}
          bool PrintAdaptiveSizePolicy                   = false                               {product}
          bool PrintCMSInitiationStatistics              = false                               {product}
          intx PrintCMSStatistics                        = 0                                   {product}
          bool PrintClassHistogram                       = false                               {manageable}
          bool PrintClassHistogramAfterFullGC            = false                               {manageable}
          bool PrintClassHistogramBeforeFullGC           = false                               {manageable}
          bool PrintCodeCache                            = false                               {product}
          bool PrintCodeCacheOnCompilation               = false                               {product}
          bool PrintCommandLineFlags                     = false                               {product}
          bool PrintCompilation                          = false                               {product}
          bool PrintConcurrentLocks                      = false                               {manageable}
          intx PrintFLSCensus                            = 0                                   {product}
          intx PrintFLSStatistics                        = 0                                   {product}
          bool PrintFlagsFinal                          := true                                {product}
          bool PrintFlagsInitial                         = false                               {product}
          bool PrintGC                                   = false                               {manageable}
          bool PrintGCApplicationConcurrentTime          = false                               {product}
          bool PrintGCApplicationStoppedTime             = false                               {product}
          bool PrintGCCause                              = true                                {product}
          bool PrintGCDateStamps                         = false                               {manageable}
          bool PrintGCDetails                            = false                               {manageable}
          bool PrintGCID                                 = false                               {manageable}
          bool PrintGCTaskTimeStamps                     = false                               {product}
          bool PrintGCTimeStamps                         = false                               {manageable}
          bool PrintHeapAtGC                             = false                               {product rw}
          bool PrintHeapAtGCExtended                     = false                               {product rw}
          bool PrintHeapAtSIGBREAK                       = true                                {product}
          bool PrintJNIGCStalls                          = false                               {product}
          bool PrintJNIResolving                         = false                               {product}
          bool PrintOldPLAB                              = false                               {product}
          bool PrintOopAddress                           = false                               {product}
          bool PrintPLAB                                 = false                               {product}
          bool PrintParallelOldGCPhaseTimes              = false                               {product}
          bool PrintPromotionFailure                     = false                               {product}
          bool PrintReferenceGC                          = false                               {product}
          bool PrintSafepointStatistics                  = false                               {product}
          intx PrintSafepointStatisticsCount             = 300                                 {product}
          intx PrintSafepointStatisticsTimeout           = -1                                  {product}
          bool PrintSharedArchiveAndExit                 = false                               {product}
          bool PrintSharedDictionary                     = false                               {product}
          bool PrintSharedSpaces                         = false                               {product}
          bool PrintStringDeduplicationStatistics        = false                               {product}
          bool PrintStringTableStatistics                = false                               {product}
          bool PrintTLAB                                 = false                               {product}
          bool PrintTenuringDistribution                 = false                               {product}
          bool PrintTieredEvents                         = false                               {product}
          bool PrintVMOptions                            = false                               {product}
          bool PrintVMQWaitTime                          = false                               {product}
          bool PrintWarnings                             = true                                {product}
         uintx ProcessDistributionStride                 = 4                                   {product}
          bool ProfileInterpreter                        = true                                {pd product}
          bool ProfileIntervals                          = false                               {product}
          intx ProfileIntervalsTicks                     = 100                                 {product}
          intx ProfileMaturityPercentage                 = 20                                  {product}
          bool ProfileVM                                 = false                               {product}
          bool ProfilerPrintByteCodeStatistics           = false                               {product}
          bool ProfilerRecordPC                          = false                               {product}
         uintx PromotedPadding                           = 3                                   {product}
         uintx QueuedAllocationWarningCount              = 0                                   {product}
         uintx RTMRetryCount                             = 5                                   {ARCH product}
          bool RangeCheckElimination                     = true                                {product}
          intx ReadPrefetchInstr                         = 0                                   {ARCH product}
          bool ReassociateInvariants                     = true                                {C2 product}
          bool ReduceBulkZeroing                         = true                                {C2 product}
          bool ReduceFieldZeroing                        = true                                {C2 product}
          bool ReduceInitialCardMarks                    = true                                {C2 product}
          bool ReduceSignalUsage                         = false                               {product}
          intx RefDiscoveryPolicy                        = 0                                   {product}
          bool ReflectionWrapResolutionErrors            = true                                {product}
          bool RegisterFinalizersAtInit                  = true                                {product}
          bool RelaxAccessControlCheck                   = false                               {product}
         ccstr ReplayDataFile                            =                                     {product}
          bool RequireSharedSpaces                       = false                               {product}
         uintx ReservedCodeCacheSize                     = 251658240                           {pd product}
          bool ResizeOldPLAB                             = true                                {product}
          bool ResizePLAB                                = true                                {product}
          bool ResizeTLAB                                = true                                {pd product}
          bool RestoreMXCSROnJNICalls                    = false                               {product}
          bool RestrictContended                         = true                                {product}
          bool RewriteBytecodes                          = true                                {pd product}
          bool RewriteFrequentPairs                      = true                                {pd product}
          intx SafepointPollOffset                       = 256                                 {C1 pd product}
          intx SafepointSpinBeforeYield                  = 2000                                {product}
          bool SafepointTimeout                          = false                               {product}
          intx SafepointTimeoutDelay                     = 10000                               {product}
          bool ScavengeBeforeFullGC                      = true                                {product}
          intx SelfDestructTimer                         = 0                                   {product}
         uintx SharedBaseAddress                         = 0                                   {product}
         ccstr SharedClassListFile                       =                                     {product}
         uintx SharedMiscCodeSize                        = 122880                              {product}
         uintx SharedMiscDataSize                        = 4194304                             {product}
         uintx SharedReadOnlySize                        = 16777216                            {product}
         uintx SharedReadWriteSize                       = 16777216                            {product}
          bool ShowMessageBoxOnError                     = false                               {product}
          intx SoftRefLRUPolicyMSPerMB                   = 1000                                {product}
          bool SpecialEncodeISOArray                     = true                                {C2 product}
          bool SplitIfBlocks                             = true                                {C2 product}
          intx StackRedPages                             = 1                                   {pd product}
          intx StackShadowPages                          = 6                                   {pd product}
          bool StackTraceInThrowable                     = true                                {product}
          intx StackYellowPages                          = 3                                   {pd product}
          bool StartAttachListener                       = false                               {product}
          intx StarvationMonitorInterval                 = 200                                 {product}
          bool StressLdcRewrite                          = false                               {product}
         uintx StringDeduplicationAgeThreshold           = 3                                   {product}
         uintx StringTableSize                           = 60013                               {product}
          bool SuppressFatalErrorMessage                 = false                               {product}
         uintx SurvivorPadding                           = 3                                   {product}
         uintx SurvivorRatio                             = 8                                   {product}
          intx SuspendRetryCount                         = 50                                  {product}
          intx SuspendRetryDelay                         = 5                                   {product}
          intx SyncFlags                                 = 0                                   {product}
         ccstr SyncKnobs                                 =                                     {product}
          intx SyncVerbose                               = 0                                   {product}
         uintx TLABAllocationWeight                      = 35                                  {product}
         uintx TLABRefillWasteFraction                   = 64                                  {product}
         uintx TLABSize                                  = 0                                   {product}
          bool TLABStats                                 = true                                {product}
         uintx TLABWasteIncrement                        = 4                                   {product}
         uintx TLABWasteTargetPercent                    = 1                                   {product}
         uintx TargetPLABWastePct                        = 10                                  {product}
         uintx TargetSurvivorRatio                       = 50                                  {product}
         uintx TenuredGenerationSizeIncrement            = 20                                  {product}
         uintx TenuredGenerationSizeSupplement           = 80                                  {product}
         uintx TenuredGenerationSizeSupplementDecay      = 2                                   {product}
          intx ThreadPriorityPolicy                      = 0                                   {product}
          bool ThreadPriorityVerbose                     = false                               {product}
         uintx ThreadSafetyMargin                        = 52428800                            {product}
          intx ThreadStackSize                           = 0                                   {pd product}
         uintx ThresholdTolerance                        = 10                                  {product}
          intx Tier0BackedgeNotifyFreqLog                = 10                                  {product}
          intx Tier0InvokeNotifyFreqLog                  = 7                                   {product}
          intx Tier0ProfilingStartPercentage             = 200                                 {product}
          intx Tier23InlineeNotifyFreqLog                = 20                                  {product}
          intx Tier2BackEdgeThreshold                    = 0                                   {product}
          intx Tier2BackedgeNotifyFreqLog                = 14                                  {product}
          intx Tier2CompileThreshold                     = 0                                   {product}
          intx Tier2InvokeNotifyFreqLog                  = 11                                  {product}
          intx Tier3BackEdgeThreshold                    = 60000                               {product}
          intx Tier3BackedgeNotifyFreqLog                = 13                                  {product}
          intx Tier3CompileThreshold                     = 2000                                {product}
          intx Tier3DelayOff                             = 2                                   {product}
          intx Tier3DelayOn                              = 5                                   {product}
          intx Tier3InvocationThreshold                  = 200                                 {product}
          intx Tier3InvokeNotifyFreqLog                  = 10                                  {product}
          intx Tier3LoadFeedback                         = 5                                   {product}
          intx Tier3MinInvocationThreshold               = 100                                 {product}
          intx Tier4BackEdgeThreshold                    = 40000                               {product}
          intx Tier4CompileThreshold                     = 15000                               {product}
          intx Tier4InvocationThreshold                  = 5000                                {product}
          intx Tier4LoadFeedback                         = 3                                   {product}
          intx Tier4MinInvocationThreshold               = 600                                 {product}
          bool TieredCompilation                         = true                                {pd product}
          intx TieredCompileTaskTimeout                  = 50                                  {product}
          intx TieredRateUpdateMaxTime                   = 25                                  {product}
          intx TieredRateUpdateMinTime                   = 1                                   {product}
          intx TieredStopAtLevel                         = 4                                   {product}
          bool TimeLinearScan                            = false                               {C1 product}
          bool TraceBiasedLocking                        = false                               {product}
          bool TraceClassLoading                         = false                               {product rw}
          bool TraceClassLoadingPreorder                 = false                               {product}
          bool TraceClassPaths                           = false                               {product}
          bool TraceClassResolution                      = false                               {product}
          bool TraceClassUnloading                       = false                               {product rw}
          bool TraceDynamicGCThreads                     = false                               {product}
          bool TraceGen0Time                             = false                               {product}
          bool TraceGen1Time                             = false                               {product}
         ccstr TraceJVMTI                                =                                     {product}
          bool TraceLoaderConstraints                    = false                               {product rw}
          bool TraceMetadataHumongousAllocation          = false                               {product}
          bool TraceMonitorInflation                     = false                               {product}
          bool TraceParallelOldGCTasks                   = false                               {product}
          intx TraceRedefineClasses                      = 0                                   {product}
          bool TraceSafepointCleanupTime                 = false                               {product}
          bool TraceSharedLookupCache                    = false                               {product}
          bool TraceSuspendWaitFailures                  = false                               {product}
          intx TrackedInitializationLimit                = 50                                  {C2 product}
          bool TransmitErrorReport                       = false                               {product}
          bool TrapBasedNullChecks                       = false                               {pd product}
          bool TrapBasedRangeChecks                      = false                               {C2 pd product}
          intx TypeProfileArgsLimit                      = 2                                   {product}
         uintx TypeProfileLevel                          = 111                                 {pd product}
          intx TypeProfileMajorReceiverPercent           = 90                                  {C2 product}
          intx TypeProfileParmsLimit                     = 2                                   {product}
          intx TypeProfileWidth                          = 2                                   {product}
          intx UnguardOnExecutionViolation               = 0                                   {product}
          bool UnlinkSymbolsALot                         = false                               {product}
          bool Use486InstrsOnly                          = false                               {ARCH product}
          bool UseAES                                    = true                                {product}
          bool UseAESCTRIntrinsics                       = true                                {product}
          bool UseAESIntrinsics                          = true                                {product}
          intx UseAVX                                    = 2                                   {ARCH product}
          bool UseAdaptiveGCBoundary                     = false                               {product}
          bool UseAdaptiveGenerationSizePolicyAtMajorCollection  = true                                {product}
          bool UseAdaptiveGenerationSizePolicyAtMinorCollection  = true                                {product}
          bool UseAdaptiveNUMAChunkSizing                = true                                {product}
          bool UseAdaptiveSizeDecayMajorGCCost           = true                                {product}
          bool UseAdaptiveSizePolicy                     = true                                {product}
          bool UseAdaptiveSizePolicyFootprintGoal        = true                                {product}
          bool UseAdaptiveSizePolicyWithSystemGC         = false                               {product}
          bool UseAddressNop                             = true                                {ARCH product}
          bool UseAltSigs                                = false                               {product}
          bool UseAutoGCSelectPolicy                     = false                               {product}
          bool UseBMI1Instructions                       = true                                {ARCH product}
          bool UseBMI2Instructions                       = true                                {ARCH product}
          bool UseBiasedLocking                          = true                                {product}
          bool UseBimorphicInlining                      = true                                {C2 product}
          bool UseBoundThreads                           = true                                {product}
          bool UseCLMUL                                  = true                                {ARCH product}
          bool UseCMSBestFit                             = true                                {product}
          bool UseCMSCollectionPassing                   = true                                {product}
          bool UseCMSCompactAtFullCollection             = true                                {product}
          bool UseCMSInitiatingOccupancyOnly             = false                               {product}
          bool UseCRC32Intrinsics                        = true                                {product}
          bool UseCodeCacheFlushing                      = true                                {product}
          bool UseCompiler                               = true                                {product}
          bool UseCompilerSafepoints                     = true                                {product}
          bool UseCompressedClassPointers               := true                                {lp64_product}
          bool UseCompressedOops                        := true                                {lp64_product}
          bool UseConcMarkSweepGC                        = false                               {product}
          bool UseCondCardMark                           = false                               {C2 product}
          bool UseCountLeadingZerosInstruction           = true                                {ARCH product}
          bool UseCountTrailingZerosInstruction          = true                                {ARCH product}
          bool UseCountedLoopSafepoints                  = false                               {C2 product}
          bool UseCounterDecay                           = true                                {product}
          bool UseDivMod                                 = true                                {C2 product}
          bool UseDynamicNumberOfGCThreads               = false                               {product}
          bool UseFPUForSpilling                         = true                                {C2 product}
          bool UseFastAccessorMethods                    = false                               {product}
          bool UseFastEmptyMethods                       = false                               {product}
          bool UseFastJNIAccessors                       = true                                {product}
          bool UseFastStosb                              = true                                {ARCH product}
          bool UseG1GC                                   = false                               {product}
          bool UseGCLogFileRotation                      = false                               {product}
          bool UseGCOverheadLimit                        = true                                {product}
          bool UseGCTaskAffinity                         = false                               {product}
          bool UseGHASHIntrinsics                        = true                                {product}
          bool UseHeavyMonitors                          = false                               {product}
          bool UseInlineCaches                           = true                                {product}
          bool UseInterpreter                            = true                                {product}
          bool UseJumpTables                             = true                                {C2 product}
          bool UseLWPSynchronization                     = true                                {product}
          bool UseLargePages                             = false                               {pd product}
          bool UseLargePagesInMetaspace                  = false                               {product}
          bool UseLargePagesIndividualAllocation        := false                               {pd product}
          bool UseLockedTracing                          = false                               {product}
          bool UseLoopCounter                            = true                                {product}
          bool UseLoopInvariantCodeMotion                = true                                {C1 product}
          bool UseLoopPredicate                          = true                                {C2 product}
          bool UseMathExactIntrinsics                    = true                                {C2 product}
          bool UseMaximumCompactionOnSystemGC            = true                                {product}
          bool UseMembar                                 = false                               {pd product}
          bool UseMontgomeryMultiplyIntrinsic            = true                                {C2 product}
          bool UseMontgomerySquareIntrinsic              = true                                {C2 product}
          bool UseMulAddIntrinsic                        = true                                {C2 product}
          bool UseMultiplyToLenIntrinsic                 = true                                {C2 product}
          bool UseNUMA                                   = false                               {product}
          bool UseNUMAInterleaving                       = false                               {product}
          bool UseNewLongLShift                          = false                               {ARCH product}
          bool UseOSErrorReporting                       = false                               {pd product}
          bool UseOldInlining                            = true                                {C2 product}
          bool UseOnStackReplacement                     = true                                {pd product}
          bool UseOnlyInlinedBimorphic                   = true                                {C2 product}
          bool UseOptoBiasInlining                       = true                                {C2 product}
          bool UsePSAdaptiveSurvivorSizePolicy           = true                                {product}
          bool UseParNewGC                               = false                               {product}
          bool UseParallelGC                            := true                                {product}
          bool UseParallelOldGC                          = true                                {product}
          bool UsePerfData                               = true                                {product}
          bool UsePopCountInstruction                    = true                                {product}
          bool UseRDPCForConstantTableBase               = false                               {C2 product}
          bool UseRTMDeopt                               = false                               {ARCH product}
          bool UseRTMLocking                             = false                               {ARCH product}
          bool UseSHA                                    = false                               {product}
          bool UseSHA1Intrinsics                         = false                               {product}
          bool UseSHA256Intrinsics                       = false                               {product}
          bool UseSHA512Intrinsics                       = false                               {product}
          intx UseSSE                                    = 4                                   {product}
          bool UseSSE42Intrinsics                        = true                                {product}
          bool UseSerialGC                               = false                               {product}
          bool UseSharedSpaces                           = false                               {product}
          bool UseSignalChaining                         = true                                {product}
          bool UseSquareToLenIntrinsic                   = true                                {C2 product}
          bool UseStoreImmI16                            = false                               {ARCH product}
          bool UseStringDeduplication                    = false                               {product}
          bool UseSuperWord                              = true                                {C2 product}
          bool UseTLAB                                   = true                                {pd product}
          bool UseThreadPriorities                       = true                                {pd product}
          bool UseTypeProfile                            = true                                {product}
          bool UseTypeSpeculation                        = true                                {C2 product}
          bool UseUTCFileTimestamp                       = true                                {product}
          bool UseUnalignedLoadStores                    = true                                {ARCH product}
          bool UseVMInterruptibleIO                      = false                               {product}
          bool UseXMMForArrayCopy                        = true                                {product}
          bool UseXmmI2D                                 = false                               {ARCH product}
          bool UseXmmI2F                                 = false                               {ARCH product}
          bool UseXmmLoadAndClearUpper                   = true                                {ARCH product}
          bool UseXmmRegToRegMoveAll                     = true                                {ARCH product}
          bool VMThreadHintNoPreempt                     = false                               {product}
          intx VMThreadPriority                          = -1                                  {product}
          intx VMThreadStackSize                         = 0                                   {pd product}
          intx ValueMapInitialSize                       = 11                                  {C1 product}
          intx ValueMapMaxLoopSize                       = 8                                   {C1 product}
          intx ValueSearchLimit                          = 1000                                {C2 product}
          bool VerifyMergedCPBytecodes                   = true                                {product}
          bool VerifySharedSpaces                        = false                               {product}
          intx WorkAroundNPTLTimedWaitHang               = 1                                   {product}
         uintx YoungGenerationSizeIncrement              = 20                                  {product}
         uintx YoungGenerationSizeSupplement             = 80                                  {product}
         uintx YoungGenerationSizeSupplementDecay        = 8                                   {product}
         uintx YoungPLABSize                             = 4096                                {product}
          bool ZeroTLAB                                  = false                               {product}
          intx hashCode                                  = 5                                   {product}
     java version "1.8.0_251"
     Java(TM) SE Runtime Environment (build 1.8.0_251-b08)
     Java HotSpot(TM) 64-Bit Server VM (build 25.251-b08, mixed mode)
     
     D:\IdeaProjects\my01>
     
     
     ```
     
     
     
     > ​	其中：-XX:+UseParallelGC 表示使用parallel scavenge GC进行垃圾回收，并且自动开启-XX:+UseParallelOldGC参数使用ParallelOldGC
     
     

详细参数请参考：	

​	Unix: https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html

​	Windows: https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html

常用参数设置：

默认情况下，java8自动开启的垃圾收集器为：parallel scavenge GC + PS MarkSweep（实现完全与Serial Old GC 一样）（以吞吐量为目标）

​	使用CMS垃圾收集器：-XX:+UseConcMarkSweepGC （ParNewGC 自动开启）（以低延迟为目标）

| 参数                    | Yong区垃圾收集器    | Old区垃圾收集器     | 描述                     |
| ----------------------- | ------------------- | ------------------- | ------------------------ |
| -XX:+UseParallelGC      | PS Scavenge         | PS MarkSweep        | 以吞吐量为目标的收集方案 |
| -XX:+UseConcMarkSweepGC | ParNew              | ConcurrentMarkSweep | 以低延迟为目标的收集方案 |
| -XX:+UseParNewGC        | ParNew              | MarkSweepCompack    |                          |
| -XX:+UseSerialGC        | Copy                | MarkSweepCompack    |                          |
| -XX:+UseParallelOldGC   | PS Scavenge         | PS MarkSweep        |                          |
| -XX:+UseG1GC            | G1 Young Generation | G1 Old Generation   |                          |
|                         |                     |                     |                          |



#### 四：内存分配与回收策略



#### 五：JDK工具



#### 六：类文件结构



#### 七：类加载机制



#### 八：字节码执行引擎



#### 九：早期编译优化（JAVAC）



#### 十：晚期编译优化（JIT）



#### 十一：Java内存模型与线程





