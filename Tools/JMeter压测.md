### 性能指标

* RT（Response Time）：响应时间，指客户端发起请求到客户端接收到服务端返回的响应，所耗费的时间
* HPS：Hits Per Second，每秒电机次数，单位是 次/秒
* TPS：Transaction Per Secon，系统每秒处理交易数，单位 笔/秒

* QPS：Query Per Second，系统每秒查询次数，单位：次/秒
* 最大响应时间：Max Response Time，用户发出请求或者指令到系统做出响应的最大时间
* 最少响应时间：Min Response Time，用户发出请求或者指令到系统做出响应的最少时间
* 90%响应时间：指所有用户的响应时间进行排序，第90%的响应时间

#### 主要的三个指标

吞吐量：每秒钟系统能处理的请求数、任务数

响应时间：服务处理一个请求或一个任务的耗时

错误率：一批请求中结果出错的请求所占比例



### jconsole 和 jvisualvm

命令行输入jconsole 可直接打开进程性能监控工具jconsole 

命令行输入jvisualvm可直接打开进程性能监控工具jvisualvm



### IDEA堆内存配置

```
-Xmx1024m // 最大堆内存
-Xms1024m // 初始堆内存
-Xmn512m  // 堆内存中的 old区 内存大小，调大可以减少old老年代的 MajorGC时间，MajorGC是MinorGC时间的十倍左右
```





