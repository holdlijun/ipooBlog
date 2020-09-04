---
title: "Java虚拟机垃圾回收-对象是否已死"
date: 2020-09-02T17:40:40+08:00
description: Java虚拟机垃圾回收-对象是否已死,如何判断对象是否存活
categories: ["Java","内存","GC"]
tags : ["Java","内存","GC","对象回收"]
featured_image:
author: "ipoo"
KeyName : "JavaGC,GC日志,垃圾回收,Java垃圾回收"
---

## 如何判断对象是否已死
在堆里存放着几乎所有的对象实例，垃圾收集器在堆进行回收前，要判断对象是否还存活着。
> Java堆是虚拟机所管理的内存中最大的一块。是所有线程共享的一块内存区域，在虚拟机启动创建。此内存区域唯一目的就是存放对象实例。

### 引用计数法（Reference counting）
在对象中添加一个计数器，每当一个地方引用它时，计数器值就加1，当引用失效时，就减一，任何时刻计数器为零的对象就不可能再被使用的。</br>
引用计数法虽然占用额外的内存空间进行计数，但是原理简单，判定效率也很高，大对数情况下都是一个不错的算法，但是在Java领域主流的虚拟机没有想用此算法管理内存，需要很多例外的情况需要考虑，譬如对象之间的相互引用的问题。

举个例子，下面代码中的testGC()方法，对象ObjA和ObjB都有instance，相互赋值，除此之外两个对象再无其他引用，实际上这两个对象已经不能在访问了，但是因为他们互相引用着对方，导致他们的引用计数都不为零，引用计数算法无法回收他们。

> [java中GC垃圾回收机制,如何查看GC日志](https://www.ipooli.com/2020/05/javagccheck/)

```txt
/**
 * @Author: www.ipooli.com
 * @DateTime: 2020/9/2 14:57
 * @Description:
 * ObjA 和 ObjB都会被回收掉，不会因为互相引用而放弃回收。
 *
 */
public class ReferenceCountingGC {
    public Object instance = null;

    private static final int _1MB = 1024 * 1024;
    /**
     * 只为占用内存，以便在GC日志看清楚是否有回收过
     */
    private byte[] bigSize = new byte[2 * _1MB];

    public static void testGC() {
        ReferenceCountingGC objA = new ReferenceCountingGC();
        ReferenceCountingGC objB = new ReferenceCountingGC();
        objA.instance = objB;
        objB.instance = objA;
        objA = null;
        objB = null;
        System.gc();

    }

    public static void main(String[] args) {
        testGC();
    }

}

```
运行结果：
```txt
[GC (System.gc()) [PSYoungGen: 7997K->840K(75776K)] 7997K->848K(249344K), 0.0012598 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 840K->0K(75776K)] [ParOldGen: 8K->630K(173568K)] 848K->630K(249344K), [Metaspace: 3122K->3122K(1056768K)], 0.0043636 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 PSYoungGen      total 75776K, used 1951K [0x000000076b600000, 0x0000000770a80000, 0x00000007c0000000)
  eden space 65024K, 3% used [0x000000076b600000,0x000000076b7e7cc0,0x000000076f580000)
  from space 10752K, 0% used [0x000000076f580000,0x000000076f580000,0x0000000770000000)
  to   space 10752K, 0% used [0x0000000770000000,0x0000000770000000,0x0000000770a80000)
 ParOldGen       total 173568K, used 630K [0x00000006c2200000, 0x00000006ccb80000, 0x000000076b600000)
  object space 173568K, 0% used [0x00000006c2200000,0x00000006c229db18,0x00000006ccb80000)
 Metaspace       used 3137K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 343K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0

```

可以看到7997K->840K ，意味着虚拟机并没有因为互相引用而放弃回收，也侧面说明虚拟机并不是通过引用计算法来判断对象的存活。


### 可达性分析算法(Reachability Analysis)

当前主流的商用程序语言的内存管理子系统，都是通过可达性分析算法来判定对象是否存活的。</br>
这个算法的基本思路就是通过一系列成为 "GC Roots" 的根对象作为起始节点集，从这些节点开始，
根据引用关系向下搜索，搜索过程所走过的路径成为"引用链",如果某个对象到GC Roots间没有任何引用链相连，
或者说从GC Roots到这个对象不可达时，则证明对象不可能再被使用的。

如图所示，对象object5，object6，object,7 虽然互有关联，但是它们到GC Roots是不可达的，因此它们会被判定为可回收的对象。
![可达性分析算法判定对象是否可回收](http://oss.ipooli.com/uploads%2F2020%2F09%2F02%2FGC%20Roots.png?Expires=1599036647)
#### GC Roots 对象包括以下几种
- 在虚拟机栈中引用的对象，比如各个线程被调用的方法堆栈中使用的参数，局部变量，临时变量等。
- 在方法区中类静态属性引用的对象。
- 在方法区中常量引用的对象，比如字符串常量池里的引用
- 在本地方法中JNI（Native方法）引用的对象
- Java虚拟机内部的引用
- 所有被同步锁（synchronized关键字）持有的对象
- 反映Java虚拟机内部情况的JMXBean，JVMTI中注册的回调，本地代码缓存等。

但是即使在可达性分析算法中判定为不可达的对象，也不是`非死不可`的,这时候会被第一次打上标记，随后在进行一次筛选，筛选的条件为此对象时候有必要执行finalize()方法。

假如对象没有覆盖finalize()方法，或者已经调用过了，那么虚拟机将这两种情况都视为"没有必要执行"。
如果这个对象被判定有必要执行finalize()方法，那么该对象将会被放置在一个名为F-Queue的队列之中，并在稍后由一条虚拟机自动建立的，低调度优先级的Finalize线程去执行它们的finalize()方法。

但不会等待它运行结束。稍后收集器会对F-Queue中的对象进行第二次小规模的标记，如果对象要在finalize()方法成功拯救自己——只要重新与任何GC Roots建立关联即可。那么在第二次标记是它会被移出"即将回收"的集合，如果对象还没逃脱，那基本上它就真的要被回收了。
从如下代码中可以看到，对象执行finalize()方法，但是它仍然可以存活。
```java
/**
 * @Author: www.ipooli.com
 * @DateTime: 2020/9/2 15:34
 * @Description: 
 *  对象可以在被GC的时候自救
 *  任何对象的finalize 只会被调用一次，如果对象面临下次的回收将不会被执行。
 */
public class FinalizeEscapeGC {
    public static FinalizeEscapeGC SAVE_HOOK = null;
    public void isAlive(){
        System.out.println("yes,i'm still alive");
    }

    /**
     * 
     * @throws Throwable
     */
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed!");
        FinalizeEscapeGC.SAVE_HOOK = this;
    }

    public static void main(String[] args) throws InterruptedException {

        SAVE_HOOK = new FinalizeEscapeGC();
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        }else {
            System.out.println("no,i'm dead");
        }
    
        //下面这段代码与上面完全相同但是这次却自救失败了
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        }else {
            System.out.println("no,i'm dead");
        }

    }

}

```

运行结果:
```txt
finalize method executed!
yes,i'm still alive
no,i'm dead
```
从结果可以看到 SAVE_HOOK对象的finalize()方法确实被垃圾收集器触发过，并且在被收集前成功逃脱了。
但是下面那段代码却自救失败了，这是一位任何一个对象的finalize()方法都只会被系统调用一次，如果对象面临下一次回收，它的finalize()方法不会被再次执行，因此第二段代码自救失败了。

但是尽量避免使用finaliz()方法，它的运行代价高昂，不确定性大，无法保证各个对象的调用顺序，如今已被官方声明为不推荐使用的语法。


## 总结
虚拟机通过可达性分析算法判定对象是否存活，对象也还有一次通过finalize()方法来自救，但不建议使用finalize()语法。


> *参考《深入理解Java虚拟机（第3版）》*: [《深入理解Java虚拟机（第3版）》](https://book.douban.com/subject/34907497/)

扫码关注公众号《ipoo》
![ipoo](http://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg)