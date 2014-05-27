最近看JMH生成的代码，发现，其类 **org.openjdk.jmh.logic.BlackHole.consumeCPU(long tokens)** 的实现
```java
    public static void consumeCPU(long tokens) {
        // If you are looking at this code trying to understand
        // the non-linearity on low token counts, know this:
        // we are pretty sure the generated assembly for almost all
        // cases is the same, and the only explanation for the
        // performance difference is hardware-specific effects.
        // Be wary to waste more time on this. If you know more
        // advanced and clever option to implement consumeCPU, let us
        // know.

        // Randomize start so that JIT could not memoize; this helps
        // to break the loop optimizations if the method is called
        // from the external loop body.
        long t = consumedCPU;

        // One of the rare cases when counting backwards is meaningful:
        // for the forward loop HotSpot/x86 generates "cmp" with immediate
        // on the hot path, while the backward loop tests against zero
        // with "test". The immediate can have different lengths, which
        // attribute to different machine code for different cases. We
        // counter that with always counting backwards. We also mix the
        // induction variable in, so that reversing the loop is the
        // non-trivial optimization.
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
这个需要抽时间看下




