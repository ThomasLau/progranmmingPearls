tomcat的log有org.apache.juli.DateFormatCache,
其中有一个类比较有意思，摘录其一处代码如下：
```java
    private String getFormat(long time) {
        long seconds = time / 1000;
        /* First step: if we have seen this timestamp 
           during the previous call, return the previous value. */
        if (seconds == previousSeconds) {
            return previousFormat;
        }
        /* Second step: Try to locate in cache */
        previousSeconds = seconds;
        int index = (offset + (int)(seconds - first)) % cacheSize;
        if (index < 0) {
            index += cacheSize;
        }
       ...
```
看代码，本意是如果时间在一秒内则直接返回，否则cache下一个时间
这似乎在一定程度上优化了？理由是日志输出是按时间递增的，且有可能是一秒内有多条...
