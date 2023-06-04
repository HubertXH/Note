### JVM Architecture

![architecture](../Resources/Image/JVM/HotSpotArchitecture.PNG "arc")

#### Run-Time Data Areas

<!-- The Java Virtual Machine defines various run-time data areas that are used during execution of a program. Some of these data areas are created on Java Virtual Machine start-up and are destroyed only when the Java Virtual Machine exits. 
Other data areas are per thread. Per-thread data areas are created when a thread is created and destroyed when the thread exits.-->    
虚拟机在运行的中会规划处不同类型的数据区域用于程序处理。有些区域会随着虚拟机的启动而创建，同时在虚拟机退出的是时候销毁。还有一些区域和线程相关(
Pre-thread)，在线程创建的时候该区域被创建，当线程停止运行退出的时候，该区域也会销毁。

##### Method Area

方法区是所有线程共享的，它存储每个类结构，如运行时常量池，字段和方法数据，以及方法和构造函数的代码，包括类和实例初始化中使用的特殊方法和接口初始化。

<!-- The Java Virtual Machine has a method area that is shared among all Java Virtual Machine threads. The method area is
analogous to the storage area for compiled code of a conventional language or analogous to the "text" segment in an
operating system process. It stores per-class structures such as the run-time constant pool, field and method data, and
the code for methods and constructors, including the special methods (§2.9) used in class and instance initialization
and interface initialization.

The method area is created on virtual machine start-up. Although the method area is logically part of the heap, simple
implementations may choose not to either garbage collect or compact it. This specification does not mandate the location
of the method area or the policies used to manage compiled code. The method area may be of a fixed size or may be
expanded as required by the computation and may be contracted if a larger method area becomes unnecessary. The memory
for the method area does not need to be contiguous.

A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of the method
area, as well as, in the case of a varying-size method area, control over the maximum and minimum method area size.

The following exceptional condition is associated with the method area: -->

    If memory in the method area cannot be made available to satisfy an allocation request, the Java Virtual Machine throws an OutOfMemoryError.    
    如果方法区内存不够时会抛出：OutOfMemoryError

##### Heap

<!-- The Java Virtual Machine has a heap that is shared among all Java Virtual Machine threads. The heap is the run-time data area from which memory for all class instances and arrays is allocated.
The heap is created on virtual machine start-up. Heap storage for objects is reclaimed by an automatic storage management system (known as a garbage collector); objects are never explicitly deallocated. 
The Java Virtual Machine assumes no particular type of automatic storage management system, and the storage management technique may be chosen according to the implementor's system requirements. 
The heap may be of a fixed size or may be expanded as required by the computation and may be contracted if a larger heap becomes unnecessary. The memory for the heap does not need to be contiguous.
A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of the heap, as well as, if the heap can be dynamically expanded or contracted, control over the maximum and minimum heap size.
The following exceptional condition is associated with the heap:
    If a computation requires more heap than can be made available by the automatic storage management system, the Java Virtual Machine throws an OutOfMemoryError.-->    
Java虚拟机的堆是所有Java虚拟机线程共享的。所有类的实例和所有数组都会被分配至Heap中。Heap在虚拟机启动的时候就会创建，用于存储那些没用被释放的对象。这块内存区域会被存储管理系统所管理例如：垃圾回收器。    
Heap的大小可以是固定值也可以是一个范围，在虚拟机运行时根据实时计算进行内存大小的分配及回收（动态分配及回收唯一的确定是
如果内存大小频分变动则会引起频繁的FullGC）。Heap在实际中可以是不连续的一段内存。    
***如果在运行过程中计算出来需要的内存大于内存管理系统所提供的内存数量，则虚拟机会抛出 OutOfMemoryError 这样的错误。***

##### Java Threads

<!-- Also call Java Virtual Machine Stacks,
Each Java Virtual Machine thread has a private Java Virtual Machine stack, created at the same time as the thread. 
A Java Virtual Machine stack stores frames (§2.6). A Java Virtual Machine stack is analogous to the stack of a conventional language such as C: it holds local variables and partial results, and plays a part in method invocation and return. 
Because the Java Virtual Machine stack is never manipulated directly except to push and pop frames, frames may be heap allocated. 
The memory for a Java Virtual Machine stack does not need to be contiguous.
In the First Edition of The Java® Virtual Machine Specification, the Java Virtual Machine stack was known as the Java stack.
This specification permits Java Virtual Machine stacks either to be of a fixed size or to dynamically expand and contract as required by the computation. 
If the Java Virtual Machine stacks are of a fixed size, the size of each Java Virtual Machine stack may be chosen independently when that stack is created.
A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of Java Virtual Machine stacks, as well as, in the case of dynamically expanding or contracting Java Virtual Machine stacks, control over the maximum and minimum sizes.
The following exceptional conditions are associated with Java Virtual Machine stacks:
    If the computation in a thread requires a larger Java Virtual Machine stack than is permitted, the Java Virtual Machine throws a StackOverflowError.
    If Java Virtual Machine stacks can be dynamically expanded, and expansion is attempted but insufficient memory can be made available to effect the expansion, or if insufficient memory can be made available to create the initial Java Virtual Machine stack for a new thread, the Java Virtual Machine throws an OutOfMemoryError.
-->
每一个Java虚拟机的线程都有一个自己私有的Stack.在线程被创建的同时线程的Stack也会被创建。Stack中主要存储
本地变量表（数组），操作数栈，运行时常量池的引用。

    如果线程栈所需的大小大于栈所被允许的最大内存空间值，则会抛出：StackOverflowError    
    如果一个线程栈的大小可以自动伸缩，若果在扩张或者新建栈的时候，没有足够的空间使用则会抛出：OutOfMemoryError    

##### Program Counter Register

<!-- The Java Virtual Machine can support many threads of execution at once (JLS §17). Each Java Virtual Machine thread has
its own pc (program counter) register. At any point, each Java Virtual Machine thread is executing the code of a single
method, namely the current method (§2.6) for that thread. If that method is not native, the pc register contains the
address of the Java Virtual Machine instruction currently being executed. If the method currently being executed by the
thread is native, the value of the Java Virtual Machine's pc register is undefined. The Java Virtual Machine's pc
register is wide enough to hold a returnAddress or a native pointer on the specific platform.    -->
程序计数器：每一个运行中的线程都有一个自己私有的程序计数器。在任意时刻每个线程执行的方法称为当前虚拟机的方法。如果当方法是不本机方法，程序计算器中包含
当前正在执行的虚拟机方法的指令地址。如果方法是本地方法，那PC中的值是未定义的。

##### Native Internal Threads

JVM 通常使用自己常规堆栈来支持本地方法（用JAVA以外的语言编写的方法）。本地方法栈也可以由JVM指令集的解析程序例如C语言来使用。

<!-- An implementation of the Java Virtual Machine may use conventional stacks, colloquially called "C stacks," to support
native methods (methods written in a language other than the Java programming language). Native method stacks may also
be used by the implementation of an interpreter for the Java Virtual Machine's instruction set in a language such as C.
Java Virtual Machine implementations that cannot load native methods and that do not themselves rely on conventional
stacks need not supply native method stacks. If supplied, native method stacks are typically allocated per thread when
each thread is created.

This specification permits native method stacks either to be of a fixed size or to dynamically expand and contract as
required by the computation. If the native method stacks are of a fixed size, the size of each native method stack may
be chosen independently when that stack is created.

A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of the native
method stacks, as well as, in the case of varying-size native method stacks, control over the maximum and minimum method
stack sizes.

The following exceptional conditions are associated with native method stacks: -->

如果一个线程需要的本地方法栈大小大于被允许的大小时（即本法方法栈计算出来的所需内存大小大于配置的最大内存），JVM会抛出StackOverflowError
错误

<!-- If the computation in a thread requires a larger native method stack than is permitted, the Java Virtual Machine throws
a StackOverflowError. -->

如果一个本地方法栈可以动态的扩展，但是没有足够的内存可供使用则 JVM 抛出OutOfMemoryError 错误

<!-- If native method stacks can be dynamically expanded and native method stack expansion is attempted but insufficient
memory can be made available, or if insufficient memory can be made available to create the initial native method stack
for a new thread, the Java Virtual Machine throws an OutOfMemoryError. -->

##### Run-Time Constant Pool

运行时常量池
    

A run-time constant pool is a per-class or per-interface run-time representation of the constant_pool table in a class
file (§4.4). It contains several kinds of constants, ranging from numeric literals known at compile-time to method and
field references that must be resolved at run-time. The run-time constant pool serves a function similar to that of a
symbol table for a conventional programming language, although it contains a wider range of data than a typical symbol
table.

Each run-time constant pool is allocated from the Java Virtual Machine's method area (§2.5.4). The run-time constant
pool for a class or interface is constructed when the class or interface is created (§5.3) by the Java Virtual Machine.

The following exceptional condition is associated with the construction of the run-time constant pool for a class or
interface:

When creating a class or interface, if the construction of the run-time constant pool requires more memory than can be
made available in the method area of the Java Virtual Machine, the Java Virtual Machine throws an OutOfMemoryError.

See §5 (Loading, Linking, and Initializing) for information about the construction of the run-time constant pool.

#### 参考文献

[Java® Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.2)