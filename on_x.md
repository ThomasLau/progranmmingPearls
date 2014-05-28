最近看JMH生成的代码，发现，其类 **org.openjdk.jmh.logic.BlackHole.consumeCPU(long tokens)** 的实现
```java
    public static void consumeCPU(long tokens) {
        // If you are looking at this code trying to understand
        // ...此处略
        long t = consumedCPU;

        // One of the rare cases when counting backwards is meaningful:
        // ...此处略
        for (long i = tokens; i > 0; i--) {
            t += (t * 0x5DEECE66DL + 0xBL + i) & (0xFFFFFFFFFFFFL);
        }

        // Need to guarantee side-effect on the result, but can't afford
        // contention; make sure we update the shared state only in the
        // unlikely case, so not to do the furious writes, but still
        // dodge DCE.
        if (t == 42) {
            consumedCPU += t;
        }
    }
```
其中似曾相识，其实是java.util.Random的部分代码，一段来源于TAOCP中算法
```java
     * This is a linear congruential pseudorandom number generator, as
     * defined by D. H. Lehmer and described by Donald E. Knuth in
     * <i>The Art of Computer Programming,</i> Volume 3:
     * <i>Seminumerical Algorithms</i>, section 3.2.1.
```
这里总结下Random的实现。
**伪随机算法
这里就不说其使用的是伪随机算法--线性同余产生随机数。TAOCP volume 3.2.1里面有对其描述，这里摘录主要部分：  
    X<sub>n+1</sub> = (aX<sub>n</sub>+c) mod m ,n>=0
此序列得到的随机序列叫做线性同余序列，并且有如下结论，  
> 当m,a,c和X<sub>0</sub>所定义的线性同余学列有周期为m的长度，当且仅当：  
> i)   c与m互素  
> ii)  对于整除m的每个素数p,b=a-1是p的倍数  
> iii) 如果m是4的倍数，则b也是4的倍数  

证明不是很难，基于原文的引理。在此不论，然而，这个结论保证了使用该算法求得的伪随机数序列是一个周期为m的序列，
并且各个[0,m)中的整数取到的概率相同，这就是可以产生伪随机序列的原理。
通常我们会将m取得足够大




