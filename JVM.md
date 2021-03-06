## 一、Java内存区域与内存溢出异常

### 1.1 运行时数据区域

![img](./jvmstructure.png)

#### 1.1.1 程序计数器(私有)

**Def：** 程序计数器是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。字节码解释器工作时就是通过改变这个计数器的值来选取下一跳需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

**作用：** 1.代码的流程控制 2.线程切换时恢复现场

**Notation：** 每条线程都有一个独立的线程计数器，唯一一个不会产生OutOfMemoryError 的内存区域。它与线程同生同死。

#### 1.1.2 虚拟机栈(私有)

**Def：**虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧(Stack Frame)用于存储局部变量表、操作数栈、动态链接、方法出口等信息，每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

**两种异常状况：** StackOverflowError和OutOfMemoryError

- **StackOverflowError：** 线程请求的深度大于虚拟机所允许的深度
- **OutOfMemoryError：** 如果虚拟机栈可以动态扩展，如果扩展时无法申请到足够的内存抛出该异常。

**Notation：** 为虚拟机执行Java方法服务。

#### 1.1.3 本地方法栈(私有)

作用类似于虚拟机栈，产生的异常也同虚拟机栈

**Notation：** 为虚拟机执行Native方法服务。

#### 1.1.4 Java 堆(线程共享)

**Def：** Java堆是所有线程共享的一块内存区域，在虚拟机启动时启动。唯一目的就是存放对象实例，几乎所有的的对象实例都在这里分配内存。是Java虚拟机所管理的内存中最大的一块。

**划分：** 新生代（Eden空间、From Survivor空间、To Survivor空间）、老生代（Tentired）

**Notation：** 垃圾收集器管理的主要区域。

**唯一异常状况：** 当堆中没存内存完成实例分配，并且堆也无法再扩展时，将抛出OutOfMemoryError异常。

#### 1.1.5 方法区(线程共享)

**作用：** 存储已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据。

**唯一异常状况：** 当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

#### 1.1.6 运行时常量池(方法区的一部分)

**Class文件组成：** 类的版本、字段、方法、接口和常量池信息

<img src="./常量池包含内容.png" alt="img" style="zoom:33%;" />

**唯一异常状况：** 当常量池无法再申请到内存时，将抛出OutOfMemoryError异常。

**Notation：** Jdk1.7之后，JVM将常量池从方法去中移了出来，再Java堆中开辟了一块区域存放运行时常量池

#### 1.1.7 直接内存(非Java内存区域的一部分)

**唯一异常状况：** 当各个内存区域总和大于物理内存限制时，将抛出OutOfMemoryError异常。

### 1.2 对象创建、内存布局、访问定位

#### 1.2.1 对象创建

<img src="./java_object_create.png" alt="image-20210317205920256" style="zoom:33%;" />

1. **类加载检查：**

虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类检查过程。

2. **分配内存：**

在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来。分配方式有“指针碰撞”和“空闲列表”两种，选择哪种分配方式有Java是否规整决定，而Java堆是否规整又由垃圾收集器是否带有压缩整理功能决定。

- **指针碰撞** 

- **空闲列表**  有点类似物理存储管理中连续内存分配

3. **初始化零值**
4. **设置对象头**

5. **执行init方法**

#### 1.2.2 对象的内存布局

- **对象头（Header）**  包括用于存储对象自身的运行时数据、指向他的类元数据的类型指针
- **实例数据**  对象真正存储的有效信息
- **对其填充**  类似c语言结构题4字节对齐

#### 1.2.3 对象的访问定位

Java栈中存储了指向对象引用的reference数据，如何通过reference操作堆上的具体对象就是对象的访问定位。主流的访问方式有使用句柄和直接指针两种。

- **句柄**

reference指向句柄池，句柄池包含指向对象实例数据的指针和指向对象类型数据的指针。

- **直接指针**

reference中存储的直接就是对象地址

### 补充：JMM（Java内存模型）

在JVM内部，Java内存模型把内存分成了两部分：线程栈区和堆区
 JVM中运行的每个线程都拥有自己的线程栈，线程栈包含了当前线程执行的方法调用相关信息，我们也把它称作调用栈。随着代码的不断执行，调用栈会不断变化。

堆区包含了Java应用创建的所有对象信息，不管对象是哪个线程创建的，其中的对象包括原始类型的封装类（如Byte、Integer、Long等等）。不管对象是属于一个成员变量还是方法中的局部变量，它都会被存储在堆区。

## 二、垃圾收集器与内存分配策略

### 2.1 概述

- **回收哪些内存**
- **什么时候回收**
- **如何回收**

### 2.2 判断对象死亡的两种方法

- **引用计数法（非虚拟机使用方法）**  有引用就加一没有引用就减一，很傻瓜，也很废
- **可达性计数法**  任何不可达GC Roots的对象被回收

原理：简单的说就是把所有的对象想像成森林如果某个树的根节点不在GC Roots上就回收这棵树。

虚拟机管理的GC Roots对象：虚拟机栈、方法区中的静态属性、方法区的常量、native方法引用的对象

### 2.3 垃圾收集算法

- **标记-清除** 

缺点：标记和清除两个过程的效率都不高；还容易引起连续分配里最让人生厌的内存碎片

- **复制**

两块内存区域，回收时复制到另一块区域，解决了内存碎片和效率问题（一个过程就解决）

内存区域的划分已变为现在虚拟机使用分代布局

- **标记-整理**
- **分代-收集**

新生代采用复制算法，老年代采用标记-整理算法。

### 2.4 垃圾收集器

**CMS（Concurrent Mark Sweep）收集器**

- 标记-清除 过程：
  - 初始标记    仅仅只标记GCRoots能直接关联到的对象（stop the world）
  - 并发标记    GCRoots Tracing
  - 重新标记    修正并发标记阶段因程序继续运行产生变动的那一部分对象的标记记录（stop the world）
  - 并发清除
- 三个缺点
  - 对CPU资源非常敏感
  - 无法处理浮动垃圾（垃圾回收过程中产生的新垃圾只能等待下一次回收
  - 标记清除算法的通病：内存碎片

**G1收集器**

- 回收过程
  - 初始标记     同上
  - 并发标记
  - 最终标记
  - 筛选回收    根据用户期望的GC停顿时间制定回收计划，根据回收所获得的空间大小以及回收所需时间的经验值，优先回收价值最大的

- 特点
  - 并行和并发
  - 分代收集
  - 空间整合    整体上采用标记整理算法，局部上采用标记复制算法
  - 可以预测的停顿

## 三、类文件结构

## 四、类加载机制

**Java语言中，类型的加载、连接和初始化过程都是在程序运行期间完成的。**

### 4.1 五个类必须加载时机

- 遇到new、getstatic、putstatic、invokestatic这四条字节码指令
- 使用java.lang.reflect包的方法对类进行反射调用
- 初始化子类时，父类没有初始化则必须初始化
- 虚拟机启动必须初始化包含main方法的类
- （这一段先抄还不理解）当使用jdk1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果是REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄🔒对应的类没有进行过初始化，则需要先出发其初始化。

**Notation：**  接口在初始化时，并不要求其父接口完成了初始化，只有在真正使用到父接口的时候才会初始化。

### 4.2 类加载的五个阶段

- **加载**

1. 通过一个类的全限定名来获取定义此类的二进制字节流
2. 将这个字节所代表的静态存储结构转化为方法区的运行时数据结构
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

- **验证**

1. 文件格式验证

   - 是否以魔数0xCAFEBABE开头

   - 主次版本号是否在当前虚拟机处理范围之类。

   - 常量池中的常量是否有不被支持的常量类型。

   - CONSTANT_Utf8_info是否有不符合UTF8编码的数据。
   - Class文件中各个部分以及文件本身是否有被删除的或附加的其他信息。
   - etc
2. 元数据验证

   - 这个类是否有父类
   - 这个类的父类是否继承了不允许被继承的类
   - 如果这个类不是抽象类，是否实现了其父类或接口之中要求实现的方法。
   - 类中的字段、方法是否与父类产生矛盾
3. 字节码验证（保证被校验的方法不会在运行时作出危害虚拟机安全的事件

   - 确保任意时刻操作数栈的数据类型与指令代码序列都能配合工作，例如操作数栈放置了一个int类型的数据，使用时却按long类型来加载入地址变量表中。
   - 保证跳转指令不会跳转到方法体以外的字节码指令上。
   - 保证方法体中的类型转换是有效的
4. 符号引用验证
   - 符号引用中通过字符串描述的全限定名是否能找到对应的类。
   - 在指定类中是否存在符合方法的字段描述符以及简单名称所描述的方法和字段。
   - 符号引用中的类、字段、方法的访问性是否可以被当前类访问。

- **准备** （正式为类变量分配内存并设置类变量初始值的阶段）

- **解析**  
- **初始化**  类构造器<clinit>
  1. 静态语句的执行与源文件的语句顺序有关，定义在之后的变量只能赋值不能读取              。
  2. 不需要显式调用父类构造器，会直接调用
  3. 父类的静态语句优于子类执行
  4. 类构造器不是必须的，要看你有没有类的函数和方法（static
  5. 接口也有类构造器，但是不用限制性父接口类构造器
  6. 虚拟机可以保证一个类的类构造器方法在多线程环境中被正确地加锁、同步

### 4.3 类加载器

- **虚拟机角度-两种不同的类加载器：**  Bootstacp ClassLoader、所有的其他的类加载器
- **开发人员角度-3种系统提供的类加载器：** 启动类加载器、扩展类加载器、应用程序加载器 

####  4.3.1 双亲委派模型

双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。父子关系不会以继承的关系来实现，使用组合关系来复用父加载器的代码。

- **过程**

如果一个类加载器收到了类加载器的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父亲加载器去完成，每一个层次的类加载器都是如此，因此所有的类加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求时，子加载器才会尝试自己去加载。

- **优点**

Java类随着它的类加载器一起具备了一种带有优先级的层次关系。使用双亲委派模型保证了类的一致性。

#### 4.3.2 破坏双亲委派

自定义构造器，重写finclass方法是接受双亲委派，重写loadclass是放弃双亲委派。