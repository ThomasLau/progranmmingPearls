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
并且各个[0,m)中的整数取到的概率相同，这就是可以产生伪随机序列的原理。通常我们会将m取得足够大。  
如下实现：new Random();
```java
public Random() {
        this(seedUniquifier() ^ System.nanoTime());
    }

    private static long seedUniquifier() {
        // L'Ecuyer, "Tables of Linear Congruential Generators of
        // Different Sizes and Good Lattice Structure", 1999
        for (;;) {
            long current = seedUniquifier.get();
            long next = current * 181783497276652981L;
            if (seedUniquifier.compareAndSet(current, next))
                return next;
        }
    }

    private static final AtomicLong seedUniquifier= new AtomicLong(8682522807148012L);
```
注意这里seed的初始化使用的是nanotime，而不是很多人以为的currenttimemillis，后者要比前者容易暴力破解多了。  
181783497276652981L这个数字其实应该是没有什么数学含义在其中，goole也没有相关信息，目前已发邮件询问，但还未有回复。  
Random还有一个支持传参seed的构造函数，其等价new()之后synchronized public void setSeed(seed)  
```java
    public int nextInt() {
        return next(32);
    }
    public int nextInt(int n) {
        if (n <= 0)
            throw new IllegalArgumentException("n must be positive");

        if ((n & -n) == n)  // i.e., n is a power of 2
            return (int)((n * (long)next(31)) >> 31);

        int bits, val;
        do {
            bits = next(31);
            val = bits % n;
        } while (bits - val + (n-1) < 0);
        return val;
    }
    protected int next(int bits) {
        long oldseed, nextseed;
        AtomicLong seed = this.seed;
        do {
            oldseed = seed.get();
            nextseed = (oldseed * multiplier + addend) & mask;
        } while (!seed.compareAndSet(oldseed, nextseed));
        return (int)(nextseed >>> (48 - bits));
    }
```
以上是nextInt()时涉及的主要方法，可以看到内部实现使用了CAS操作，即自旋锁。这也可见Random是**Thread-safe**的，  
事实的确如此，可以在Math.random()中看到，其提供的静态方法正式依据private static Random randomNumberGenerator的实现  
然而，同样可见的是，Random虽然是thread-safe的，但已知是cas在线程多起来的时候效果不是很好，并且多线程共享一个Random  
是由问题的，因为他们公用一个随机序列（seed是一样的）  
所以java 1,7中提供了一个较好的实现--**ThreadLocalRandom extends Random**  
（看起源码发现其还使用pad0...来避免 **CPU的缓存竞争** 这个后面会写一篇专门讨论）  
使用：java.util.concurrent.ThreadLocalRandom.current().nextInt(10)


