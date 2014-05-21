damn headache,why this behave unexpected??
```java
public final class FalseSharingV2 implements Runnable {
	public final static int NUM_THREADS = 4; // change
	//private static VolatileLong[] longs = null;//new VolatileLong[NUM_THREADS];
	private final static VolatileLong[] longs =new VolatileLong[NUM_THREADS];
	public /*final*/ static long ITERATIONS = 500L * 1000L * 1000L;
	//private final static long ITERATIONS2 = 500L * 1000L * 1000L;
	private final int arrayIndex;
	
//	static {
//		for (int i = 0; i < longs.length; i++) {
//			longs[i] = new VolatileLong();
//		}
//	}
	static void init(){
		//longs = new VolatileLong[NUM_THREADS];
		for (int i = 0; i < longs.length; i++) {
			longs[i] = new VolatileLong();
		}
	}

	public FalseSharingV2(final int arrayIndex) {
		this.arrayIndex = arrayIndex;
	}

	public static void main(final String[] args) throws Exception {
		new FalseSharingV2(1);
		init();
		for (int j = 1; j < 5; j++) {
			//NUM_THREADS=j;
			//init();
			final long start = System.nanoTime();
			runTest();
			System.out.println("duration = " + (System.nanoTime() - start));
		}
	}

	private static void runTest() throws InterruptedException {
		Thread[] threads = new Thread[NUM_THREADS];
		for (int i = 0; i < threads.length; i++) {
			threads[i] = new Thread(new FalseSharingV2(i));
		}
		for (Thread t : threads) {
			t.start();
		}

		for (Thread t : threads) {
			t.join();
		}
	}

	public void run() {
		//long i = ITERATIONS + 1;
		long i=500L * 1000L * 1000L+1;
		while (0 != --i) {
			longs[arrayIndex].value = i;
		}
	}

	public final static class VolatileLong {
		public volatile long value = 0L;
		public long p1, p2, p3, p4, p5, p6; // comment out
	}
}
```
this output will be:
···java
duration = 21278060524
duration = 21110399588
duration = 21325678304
duration = 21329919433
···