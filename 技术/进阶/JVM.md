# JVM内存区域

## 运行时数据区域

Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域有各自的用途，以及创建和销毁的时间， 有的区域随着虚拟机进程的启动而一直存在，有些区域则是依赖用户线程的启动和结束而建立和销毁。

![202303021259740](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071615566.jpg)

### 程序计数器（`Program Counter Register  `）

可以看作是**当前线程所执行的字节码的行号指示器**。字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。

同一个时间，一个处理器（或一个内核）都只会执行一条指令，因此每条线程都有一个独立的程序计数器，各线程间计数器互不影响。是**线程私有**的内存。

如果线程正在执行一个Java方法，那么计数器记录的是正在执行的虚拟机字节码指令的地址；如果执行的是本地方法，那么计数器值为空（`Undefined`）

### Java虚拟机栈（`Java Virtual Machine Stack  `）

生命周期与线程相同，是**线程私有**的。

每个方法被执行的时候，Java虚拟机都会同步创建一个栈帧，用于存储局部变量表、操作数栈、动态连接、方法出口等信息。每一个方法被调用直至执行完毕的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

局部变量表存放了**编译期**可知的各种基本数据类型（`boolean`、 `byte`、 `char`、 `short`、 `int`、`float`、 `long`、 `double`）、对象引用（`reference`类型， 它并不等同于对象本身， 可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或者其他与此对象相关的位置）和`returnAddress`类型（指向了一条字节码指令的地址）。

> 在《Java虚拟机规范》 中， 对这个内存区域规定了两类异常状况： 如果线程请求的栈深度大于虚拟机所允许的深度， 将抛出`StackOverflowError`异常； 如果Java虚拟机栈容量可以动态扩展， 当栈扩展时无法申请到足够的内存会抛出`OutOfMemoryError`异常。  

### 本地方法栈（`Native Method Stacks  `）

本地方法栈（Native Method Stacks） 与虚拟机栈所发挥的作用是非常相似的， 其区别只是虚拟机栈为虚拟机执行Java方法（也就是字节码） 服务， 而本地方法栈则是为虚拟机使用到的本地（Native）方法服务。  

### Java堆（`Java Heap`）

Java堆是虚拟机所管理的内存中最大的一块。是**被所有线程共享**的一块内存区域，在虚拟机启动时创建。**唯一目的就是存放对象实例，几乎所有对象实例都在这里分配内存。**

Java堆是垃圾收集器管理的内存区域，因此一些资料中也称作为“GC堆”。从回收内存的角度看，由于现代垃圾收集器大部分都是基于分代收集理论设计的，所以 Java 堆还可以细分为：新生代和老年代；再细致一点有：Eden、Survivor、Old 等空间。进一步划分的目的是更好地回收内存，或者更快地分配内存。

Java堆可以处于物理上不连续的内存空间中， 但在逻辑上它应该被视为连续的。

### 方法区（`Method Area  `）

与Java堆一样，是线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

垃圾收集行为在这个区域比较少出现，该区域的内存回收目标主要是针对常量池的回收和对类型的卸载。

### 运行时常量池（`Runtime Constant Pool  `）

运行时常量池是方法区的一部分。Class文件中除了有类的版本、 字段、 方法、 接口等描述信息外， 还有一项信息是常量池表（Constant Pool Table） ， 用于存放编译期生成的各种字面量与符号引用， 这部分内容将在类加载后存放到方法区的运行时常量池中。  

字面量是源代码中的固定值的表示法，即通过字面我们就能知道其值的含义。字面量包括整数、浮点数和字符串字面量，符号引用包括类符号引用、字段符号引用、方法符号引用和接口方法符号引用。

常量池表会在类加载后存放到方法区的运行时常量池中。

运行时常量池的功能类似于传统编程语言的符号表，尽管它包含了比典型符号表更广泛的数据。

### 字符串常量池

**字符串常量池** 是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

### 直接内存（`Direct Memory  `）

直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 `OutOfMemoryError` 错误出现。

JDK1.4 中新加入的 **NIO(New Input/Output) 类，引入了一种基于通道（Channel）与缓存区（Buffer）的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 `DirectByteBuffer` 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为\**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

## `HotSpot`虚拟机对象探秘

### 对象的创建

#### 第一步：类加载检查

当虚拟机遇到一条字节码new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载、解析和初始化过。如果没有，必须先执行相应的类加载过程

#### 第二步：分配内存

对象所需内存的大小在类加载完成后便可以完全确定，为对象分配空间的人物实际上便等同于把一块确定大小的内存块从Java堆中划分出来。有两种分配方式**指针碰撞**和**空闲列表**。选择哪种分配方式有Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有空间压缩整理能力而决定。

- 指针碰撞
  - 内存是绝对规整的
  - 所有被使用过的内存都被放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲的方向挪动一段与对象大小相等的距离。
- 空闲列表
  - 内存不是规整的，已被使用的内存和空闲的内存相互交错在一起
  - 虚拟机必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录。

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的。

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程安全，通常来讲，虚拟机采用两种方式来保证线程安全：

- `CAS`+失败重试：CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- `TLAB`：为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

#### 第三步：初始化为零值

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头）， 这一步操作保证了对象的实例字段在Java代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

#### 第四步：设置对象头

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

#### 第五步：执行`init`方法

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

### 对象的内存布局

对象在堆内存中存储布局可以划分三个部分：对象头、实例数据和对齐填充

**Hotspot 虚拟机的对象头包括两部分信息**，**第一部分用于存储对象自身的运行时数据**（哈希码、GC 分代年龄、锁状态标志等等），**另一部分是类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

实例数据部分是对象真正存储的有效信息。

对其填充，这并不是必然存在的，也没有特别的含义，仅仅起着占位符的作用。由于`HotSpot`虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，也就是说任何对象大小都必须是8字节的整数倍，因此如果对象实例数据部分没有对齐的话，就需要通过对齐填充来补全。

### 对象的访问定位

创建对象是为了使用对象。Java程序会通过栈上的`reference`数据来操作对上的具体对象。主流的访问方式主要有**使用句柄**和**直接指针**两种。

- 使用句柄，Java堆中将可能会划分处一块内存来作为**句柄池**，`reference`中的存储就是对象的句柄地址，句柄包含了对象实例数据与类型数据各自具体的地址信息。
- 直接指针，reference 中存储的直接就是对象的地址。

这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。

![202303021414552](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071616639.jpg)

![202303021415181](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071616143.jpg)

# JVM垃圾回收

## 死亡对象判断方法

### 引用计数算法

在对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加一；当引用失败时，计数器值就减一；任何时刻计数器值为零的对象就是不可能再被使用的。

目前主流的虚拟机并没有选择这个算法来管理内存，最主要原因是她很难解决对象之前相互循环引用的问题。

### 可达性分析算法

通过一系列称为`GC Roots`的跟对象作为起始节点集，从这些节点开始，根据引用关系乡下搜索，搜索过程所走的路径称为**引用链**，如果某个对象到`GC Roots`间没有任何引用链相连，则证明此对象是不可能在被使用的。

![202303021434147](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071616025.jpg)

固定可以作为`GC Roots`的对象包括：

- 栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中引用的对象
- 被同步锁持有的对象

在可达性分析算法中判定为不可达对象，并不一定必须回收。

至少要经历两次标记过程：发现没有与`GC Roots`相连接，会被第一次标记并进行一次筛选，筛选条件是次对象是否有必要执行`finalize`方法。当对象没有覆盖`finalize`方法，或已被调用过（`finalize`方法智慧被系统自动调用一次），虚拟机将这两种情况视为没有必要执行。

被判定为需要执行的对象将会被放在一个队列中进行第二次标记，除非这个对象与引用链上的任何一个对象建立 关联，否则会被真的回收。

### 引用类型

- 强引用（`Strongly Reference`），数程序中最普遍存在的引用赋值，垃圾收集器永远不会回收掉被强引用的对象
- 软引用（`Soft Reference`），只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围进行第二次回收。
- 弱引用（`Weak Reference`），弱引用关联的对象智能生存到下一次垃圾收集发生为止。当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。
- 虚引用（`Pahntom Reference`），是最弱的一种引用关系。一个对象是否有虚引用存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。唯一目的知识为了能在这个对象被收集器回收时收到一个系统通知。

### 回收方法区

方法去的垃圾收集主要回收两部分内容：废弃的常量和不再使用的类型

#### 废弃的常量

如果字符串常量池中存在"java"常量，但是当前系统没有任何一个字符串对象引用该常量，且虚拟机中也没有其他地方引用这个字面量。那么在发生内存回收，而且收集器判断有必要的话，这个常量就会被系统清理出常量池。

#### 不再使用的类型

需要同时满足以下三个条件，才被判定为**不再使用的类型**

- 该类所有实例都已经被回收
- 加载该类的类加载器已经被回收
- 该类对应的`java.lang.Class`对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

## 垃圾收集算法

从如何判定对象消亡的角度出发，垃圾收集算法可以划分为**引用技术式垃圾收集**和**追踪式垃圾收集**

- 部分收集（`Partial GC`），不是完整收集整个Java堆的垃圾收集
  - 新生代收集（`Minor GC \ Young GC`），新生代的垃圾收集
  - 老年代收集（`Major GC \ Old GC`），老年代的垃圾收集
  - 混合收集（`Mixed GC`），收集整个新生代以及部分老年代的垃圾收集
- 整堆收集（`Full GC`），收集整个Java堆和方法区的垃圾收集

### 分代收集算法

收集器将Java堆划分出不同的区域，然后将回收对象依据其年龄（即对象熬过垃圾收集过程的次数）分配到不同的区域之中存储。

在新生代中，每次垃圾收集时都发现有大批对象死去，而每次回收后存活的少量对象，将会逐步晋升到老年代中存放。

但现实情况中，对象不是孤立的，对象之间会存在跨代引用。在新生代建立一个全局的数据结构**记忆集**，把老年代划分为若干小块，标识处老年代的哪一块内存会存在跨代引用，只有包含了跨代引用的小块内的内存里的对象才会被加入到`GC Roots`进行扫描。避免了为了找出跨代引用而扫描整个老年代。

### 标记-清除算法

算法分为**标记**和**清除**两阶段，首先标记处所有需要回收的对象，在标记完成后，统一回收掉所有被标记的对象；或者标记存活的对象，统一回收未被标记的对象。

**缺点**

- 执行效率不稳定，Java堆中包含大量对象，其中大部分是需要被回收的，这时需要进行大量标记和清除的动作，导致效率都随数量增长而降低
- 内存空间的碎片化问题，清除后会产生大量**不连续的内存碎片**，空间碎片太多可能会导致当以后在程序运行中需要分配较大对象时无法找到足够的连续内存而不得不提前出发一次垃圾收集行为。

![image-20230302152407257](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071616382.png)

### 标记-复制算法

为了解决标记-清除算法面对大量可回收对象时执行效率低的问题。

它将可用内存划分为**大小相等**的两块，每次只使用其中一块。当这一块内存用完了，就将还存活的对象复制到另一块上面，然后再把已使用的内存空间一次清理掉。如果内存中多数对象都是存活的， 这种算法将会产生大量的内存间复制的开销， 但对于多数对象都是可回收的情况， 算法需要复制的就是占少数的存活对象， 而且每次都是针对整个半区进行内存回收， 分配内存时也就不用考虑有空间碎片的复杂情况， 只要移动堆顶指针， 按顺序分配即可。

代价是将可用内存缩小为了原来的一般，空间浪费太多。

![image-20230302152637457](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071617356.png)

大多虚拟机有限采用了这种收集算法去回收**新生代**

一种优化的半区复制分代策略：

把新生代分为一块较大的`Eden`空间和凉快较小的`Survivor`空间，每次分配内存只使用`Eden`和一块`Survivor`，发生垃圾收集时，将`Eden`和`Survivor`中仍存活的对象一次性复制到另一块`Survivor`中，直接清理掉`Eden`和已用过的`Survivor`空间

### 标记-整理算法

针对老年代对象的特征提出的一种算法。标记过程仍然与标记-清除一样，但后续步骤是让所有存活的对象都向空间一端移动，然后直接清理掉边界以为的内存。

![image-20230302153231748](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071617333.png)

## `HotSpot`算法细节

### 记忆集和卡表

记忆集是一种用于记录从非收集区指向收集区的指针集合的抽象数据结构。粒度不必太细，否则会造成巨大的空间占用和维护成本。

其中**卡精度**记录精确到每一块内存区域，该区域内有对象含有跨代指针。称为卡表。卡表内每一个元素都对应着一块特定大小的内存块，称为卡页。只要卡页内有一个或以上对象的字段存在跨代指针，就将对应卡表值标识为1，称为元素变脏。只需要筛选出卡表变脏的元素，就能轻易得出哪些卡页内存包含跨代指针，把他们加入`GC Roots`中一并扫描。

### 写屏障

卡表变脏的时间：有其他分代区域中对象引用了本区域对象时，对应卡表就应该变脏。

`HotSpot`通过写屏障维护卡表。

### 并发的可达性分析

解决兵法扫描时对象消失问题，有两种方案：

- 增量更新，当已扫描对象插入新的指向未扫描对象的引用关系时，将这个新插入的引用记录下来，等兵法扫描结束后，再将这些及路过的引用关系中的已扫描对象作为根，重新扫描。
- 原始快照，当已访问过但未完全扫描的对象A删除指向未扫描对象B的引用关系时，就将这个要删除的引用记录下来，并发扫描结束之后，再将这些及路过的引用关系中的A为根重新扫描一次。

## 垃圾收集器

### `Serial`

是最基础的收集器，是一个单线程工作的收集器。它的 **“单线程”** 的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作时，必须暂停其他所有工作线程，直到他收集完成。

他相比于其他收集器的单线程，优势是简单而高效。没有线程交互的开销，额外的内存消耗最小。

Serial对于运行在Client的虚拟机来说是很好的选择。

![202303021600174](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071617962.jpg)

### `ParNew`

`ParNew`实质上是`Serial`的多线程并行版本。除了使用多线程进行垃圾收集外，其余行为与`Serial`完全一致。

![202303021602771](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071617845.jpg)

**并行和并发概念补充：**

- **并行（Parallel）** ：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。
- **并发（Concurrent）**：指用户线程与垃圾收集线程同时执行（但不一定是并行，可能会交替执行），用户程序在继续运行，而垃圾收集器运行在另一个 CPU 上。

### `Parallel Scavenge`

是一款新生代收集器，基于复制算法实现的收集器，也是能够并行收集的多线程收集器。

`Parallel Scavenge`关注的是吞吐量（高效利用CPU），而`CMS`等收集器关注点更多是用户线程的停顿时间。**所谓吞吐量就是 CPU 中用于运行用户代码的时间与 CPU 总消耗时间的比值。**

### `Serial Old`

**Serial 收集器的老年代版本**，它同样是一个单线程收集器。它主要有两大用途：一种用途是在 JDK1.5 以及以前的版本中与 Parallel Scavenge 收集器搭配使用，另一种用途是作为 CMS 收集器的后备方案。

### `Parallel Old`

**Parallel Scavenge 收集器的老年代版本**。使用多线程和“标记-整理”算法。在注重吞吐量以及 CPU 资源的场合，都可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。

### `CMS - Concurrent Mark Sweep`

是一种以获取最短回收停顿时间为目标的收集器，从名字上可以看出是基于标记-清除算法实现的。包括四个步骤：

1. 初始标记：仅仅只是标记一下`GC Roots`能直接关联到的对象，速度很快，但需要暂停用户线程
2. 并发标记：就是从`GC Roots`的直接关联对象开始遍历整个对象图的过程，这个过程耗时长但不需要停顿用户线程，可以与垃圾收集线程一起并发运行。
3. 重新标记：为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，采用增量更新的方法。需要暂停用户线程。
4. 并发清除：清理删除掉标记阶段判断已经死亡的对象，由于不需要移动存活的对象，这个阶段可以与用户线程并发进行。

![202303021618526](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071618236.jpg)

**优点**：并发收集、低停顿

**缺点**：

- 对CPU资源非常敏感
- 无法处理浮动垃圾，由于用户线程是还在继续运行的，程序在运行自然会不断产生垃圾，但这一部分垃圾对象是出现在标记过程结束以后，`CMS`无法在当次收集处理他们。如果在运行期间预留的内存无法满足程序分配新对象的需求，就会出现一次**并发失败**，导致一次暂停用户线程而启动`Serial Old`收集器来进行老年代的垃圾收集
- 由于基于标记-清除算法实现，会产生大量空间碎片。

### `G1 - Garbage First`

是一款面向服务器的垃圾收集器，主要针对配备多颗处理器及大容量内存的机器。以极高概率满足 GC 停顿时间要求的同时，还具备高吞吐量性能特征。

`G1`可以面向堆内存任何部分来组成回收集进行回收，衡量标准不再是它属于哪个分代，而是哪块内存中存放的垃圾数量最多，回收收益最大，这就是`G1`收集器的`Mixed GC`模式。

`G1`开创基于`Region`的堆内存布局是它能够实现这个目标的关键。不再坚持固定大小以及固定数量的分代区域划分，而是把连续的Java堆划分为多个大小相等的独立区域，每个`Region`都可以根据需要，扮演新生代的`Eden`、`Survivor`或者老年代空间。`Region`中还有一类特殊的`Humongous`区域，专门用来存储大对象。

**步骤**

1. 初始标记，仅仅只是标记一下GC Roots能直接关联到的对象  
2. 并发标记，从GC Root开始对堆中对象进行可达性分析， 递归扫描整个堆里的对象图， 找出要回收的对象， 这阶段耗时较长， 但可与用户程序并发执行  
3. 最终标记，对用户线程做另一个短暂的暂停，通过**原始快照**处理并发阶段结束后仍遗留下来的最后那少量的SATB记录。  
4. 筛选回收，`G1`收集器跟踪各个`Region`里面的垃圾堆积的价值大小，在后台维护一个优先级列表。优先处理回收价值收益最大的那些`Region`。把决定回收那一部分`Region`的存活对象复制到空的`Region`中，再清理掉整个旧`Region`的全部空间。由于涉及存活对象的移动，必须暂停用户线程。

`G1`从整体上看是基于标记-整理算法实现的，但从局部（两个`Region`之间）来看，又是基于标记-复制算法实现的。无论如何，都不会产生空间碎片，收集完毕后能提供规整可用的内存空间。

### `Shenandoah`

### `ZGC - Z Garbage Collector`

希望在尽可能对吞吐量影响不太大的前提下，实现在任意堆内存大小下都可以把垃圾收集的停顿实践限制在十毫秒以内的低延迟。

是一款基于`Region`的堆内存布局，不设置分代，使用了读屏障、染色指针和内存多重映射等技术来实现可并发的标记-整理算法。

#### 动态`Region`

`ZGC`的`Region`具有动态性——动态创建和销毁，以及动态的区域容量大小：

- 小型：固定容量为`2MB`，用于放置小于`256KB`的小对象
- 中型：固定容量为`32MB`，用于放置大于等于`256KB`但小于`4MB`的对象
- 大型：容量不固定，可以动态变化，但必须为2MB的整数倍，用于放置`4MB`或以上的大对象，每个大型`Region`只存放一个大对象。

#### 染色指针技术

`ZGC`通过染色指针技术实现并发整理算法：将少量额外的信息存储在指针上。

#### 运作过程

![202303021754254](https://mervyn-markdown-images.oss-cn-beijing.aliyuncs.com/202311071618059.jpg)

- 并发标记（`Concurrent Mark`）：便利对象图做可达性分析的阶段，前后也要经过初始标记、最终标记的短暂停顿。`ZGC`的标记是在指针上而不是在对象上。
- 并发预备重分配（`Concurrent Prepare for Relocate`）：需要根据特定的查询条件统计得出本次收集过程要清理哪些`Region`，将这些`Region`组成重分配组。`ZGC`每次回收都会扫描所有的`Region`，用范围更大的扫描成本换取省去`G1`记忆集的维护成本。因此重分配集只决定了里面的存活对象会被复制到其他`Region`中，里面的`Region`会被释放
- 并发重分配（`Concurrent Relocate`）：是`ZGC`的核心阶段。这个过程要把重分配集中的存活对象复制到新的`Region`中，并为重分配集中的每个`Region`维护一个**转发表**，记录从旧对象到新对象的转向关系。如果用户线程此时并发访问了位于重分配集中的对象，这次访问将会被预置的内存屏障所截获，然后立即根据`Region`的转发表记录将访问转发到新复制的对象上，并同时修正更新该引用的值，使其直接指向新对象，`ZGC`称这种行为为指针的**自愈**能力。还有另一个好处就是由于染色指针的存在，一旦重分配集中某个`Region`的存活对象都复制完后，该`Region`就可以立即释放用于新对象的分配（转发表还得留着不能释放），哪怕堆中还有很多指向这个对象的为更新指针也没关系，这些旧指针一旦被使用，都是可以**自愈**的。
- 并发重映射（`Concurrent Remap`）：重映射做的就是修正整个堆中指向重分配集中旧对象的所有引用，但这并不是迫切需要完成的任务。重映射目的主要是为了减少旧指针第一次访问时多一次的转发和修正，及清理后可以释放转发表带来的收益。由于并不迫切，所有`ZGC`将该工作合并到下一次垃圾收集循环中的**并发标记**阶段里完成，反正都要便利所有对象，这样就节省了一次便利对象图的开销。

#### 劣势

`ZGC`完全没有使用记忆集，它甚至连分代都没有，卡表也不需要，完全没有用到写屏障。这样的选择限制了它承受的对象分配速率不会太高。`ZGC`对很大的堆做一次完整的并发收集，在折断时间里，由于应用的对象分配速率很高，将创造大量的新对象，这些新对象很难进入本次收集的标记范围，通常就只能当作存活对象看待，这就产生了大量的浮动垃圾。这种情况持续进行的话，堆中剩余可腾挪的空间越来越小。

## 内存分配和回收策略

### 对象优先在`Eden`区分配

大多数情况下，对象在新生代中 Eden 区分配。当 Eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC。

### 大对象直接进入老年代

大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。

大对象直接进入老年代主要是为了避免为大对象分配内存时由于分配担保机制带来的复制而降低效率。

### 长期存活的对象将进入老年代

对象通常在Eden区里诞生， 如果经过第一次Minor GC后仍然存活， 并且能被Survivor容纳的话， 该对象会被移动到Survivor空间中， 并且将其对象年龄设为1岁。 对象在Survivor区中每熬过一次Minor GC， 年龄就增加1岁， 当它的年龄增加到一定程度（默认为15） ， 就会被晋升到老年代中。

### 动态对象年龄判定

如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代。

### 空间分配担保

空间分配担保是为了确保在 Minor GC 之前老年代本身还有容纳新生代所有对象的剩余空间。

> 在发生 Minor GC 之前，虚拟机必须先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那这一次 Minor GC 可以确保是安全的。如果不成立，则虚拟机会先查看 `-XX:HandlePromotionFailure` 参数的设置值是否允许担保失败(Handle Promotion Failure);如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试进行一次 Minor GC，尽管这次 Minor GC 是有风险的;如果小于，或者 `-XX: HandlePromotionFailure` 设置不允许冒险，那这时就要改为进行一次 Full GC。
>
> JDK 6 Update 24 之后的规则变为只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小，就会进行 Minor GC，否则将进行 Full GC。

# 参考资料

[深入理解Java虚拟机（第3版） (豆瓣) (douban.com)](https://book.douban.com/subject/34907497/)

[JavaGuide（Java学习&&面试指南） | JavaGuide(Java面试+学习指南)](https://javaguide.cn/home.html#jvm-必看)