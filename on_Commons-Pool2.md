待有时间找个格式化代码的方法....markdown似乎不是好用啊...while, another markup language...

闲来无事看代码，今看到如下，先贴一段commons-pool2里面的test代码:
1,
org.apache.commons.pool2.impl.TestSoftRefOutOfMemory.testOutOfMemory1000()这个方法
```java
    pool = new SoftReferenceObjectPool<String>(new SmallPoolableObjectFactory());
    for (int i = 0 ; i < 1000 ; i++) {
	    pool.addObject();
    }
    String obj = pool.borrowObject();
    pool.returnObject(obj);
    obj = null;
    assertEquals(1000, pool.getNumIdle());
    final List<byte[]> garbage = new LinkedList<byte[]>();
    final Runtime runtime = Runtime.getRuntime();
    while (pool.getNumIdle() > 0) {
    	try {
    		long freeMemory = runtime.freeMemory();
    		if (freeMemory > Integer.MAX_VALUE) {
    			freeMemory = Integer.MAX_VALUE;
    		}
    		System.out.println("size:"+garbage.size()+"\tmem:"+runtime.freeMemory()
    		    +"\tidle:"+pool.getNumIdle()+"\tactive:"+pool.getNumActive());
    		garbage.add(new byte[Math.min(1024 * 1024, (int)freeMemory/2)]);
    	} catch (OutOfMemoryError oome) {
    		System.out.println(oome);
    		//System.gc();
    	}
    	//System.gc();
    }
    garbage.clear();
    System.gc();
```
源代码意思显然是期待利用SoftReferenceObjectPool回收的特性

> SoftReference:
All soft references to softly-reachable objects are guaranteed to have been cleared 
before the virtual machine throws an OutOfMemoryError.

但是跑了一下代码，发现事与愿违啊.程序进入死循环了.

难道该常识有误？显然不是这样的，pool里的对象在OOM时候应该被回收的,问题就出在pool.getNumIdle()这一句，追踪代码最后到如下：
org.apache.commons.pool2.impl.SoftReferenceObjectPool.removeClearedReferences
	
```java
    PooledSoftReference<T> ref;
	while (iterator.hasNext()) {
		ref = iterator.next();
		if (ref.getReference() == null || ref.getReference().isEnqueued()) {
			iterator.remove();
		}
	}
```
这里的bug涉及到没有区分强弱引用问题，待有空细述.不过这里可以在if里判断加ref.getReference().get() == null来避免该bug
理由是：SoftReferenceObjectPool使用的是PooledSoftReference对象，PooledSoftReference has-a SoftReference,这是一个强引用，OOM前回收时，强引用是不胡回收的，这里判断ref.getReference() == null || ref.getReference().isEnqueued()来打到remove**应是**不对的

2,PerformanceTest
  单从示例给的数据，是符合预期的。但奇怪的是，修改了sleep时间，甚至设置为0，测算出的性能表现不是预期，甚至相悖甚远，待探明。
