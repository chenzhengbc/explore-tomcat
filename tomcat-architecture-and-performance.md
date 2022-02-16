## Tomcat Architecture

![tomcat-architecuture](https://raw.githubusercontent.com/chenzhengbc/md-images/develop/uPic/tomcat-architecture.png)

## Tomcat Connector

#### NIO 

```xml
<Connector port="8080" protocol="org.apache.coyote.http11.Http11NioProtocol">
</Connector>
```

- handle multiple connections per thread by using poller threads to keep connections alive
- and worker threads to process incoming requests. 
- NIO2 builds upon NIO and includes additional features for handling asynchronous I/O operations.

### Request and worker threads

Each connector manages its workload with a pool of **worker** threads and one or more **acceptor** threads.

#### Acceptor Thread

- The **acceptor** thread **asssigns** the **connection** to an avaialble **worker** thread from the pool
- Then goes back to listening for new connections 
- When a connector receives a reqeust from a client.

#### Worker Thread

- Send request to the engine
- which **processes** the **request** and creates the appropriate **response** based on the request headers and associated virtual host and context
- It becomes available again once the engine returns a response for the connector to transmit back to the client

## Tomcat Threads

### Type of Thread

#### Poller Thread 

Poller thread is used to keep connection alive

#### Worker Thread

Processing incoming requests

### Thread Configuration

The number of threads in the pool depends on the parameters you’ve set for the connector in your **conf/server.xml** file. 

- `minSpareThreads`: Default 10. The minimum number of threads that should be running at all times. This includes idle and active threads. It is the initial threads tomcat creates at start up, up to the number of `maxThreads`
- `maxThreads`: Default 200. It is the maximum number of **threads allowed to run** at any given time. It is the number of **worker threads** that are available.
- `maxConnections`: Default 10,000. The total number o**f concurrent connections** that the server will **accept** and process. Any **additional** incoming connections will be placed in a **queue** until a thread becomes available. 
- `acceptCount`: Default 100. the maximum number of TCP requests that can **wait in a queue** at the **OS level** when there are no worker threads available. 
- `connectionTimeout`: Default 20,000. The number of milliseconds a connector will **wait** **before closing an IDLE connection**.  

Note:

- When the `maxThreads` is reached, the server will only continue to accept **up to** `maxConnections` number of concurrent connections. 

- Once the limit is reached, the new connections are placed in `acceptCount` queue to wait for next avaialble thread.

- When the # of connections reaches `maxConnections` and the queue is full, `Connection Refused` error will be returned. 

- An **increase in processing time** without a corresponding spike in the number of requests could indicate that

  - **your server does not have enough worker threads** available to process requests 
  - or a complex database query is slowing down the server. 

- If processing time increases as the traffic to your server increases, you can start addressing this issue by first increasing the number of **`maxThreads`** available for a connector, which will increase the number of worker threads that are available to process requests. 

- If you still notice slow request processing times after you’ve increased the `maxThreads` parameter, then your server’s **hardware may not be equipped** to manage the growing number of worker threads processing incoming requests. In this case, you may need to **increase server memory or CPU**.

- f the executor queue is full and can’t accept another incoming request. You will see an entry in Tomcat’s server log (**/logs/Catalina.XXXX-XX-XX.log**) similar to the following example:

  ```xml
  WARNING: Socket processing request was rejected for: <socket handle>
  java.util.concurrent.RejectedExecutionException: Work queue full.
    at org.apache.catalina.core.StandardThreadExecutor.execute
    at org.apache.tomcat.util.net.(Apr/Nio)Endpoint.processSocketWithOptions
    at org.apache.tomcat.util.net.(Apr/Nio)Endpoint$Acceptor.run
    at java.lang.Thread.run
  ```



# Tomcat Monitor

## Metric to watch

### currentThreadsBusy/activeCount

The **currentThreadsBusy** (ThreadPool) and **activeCount** (Executor) metrics tell you how many threads from the connector’s pool are currently processing requests. 

With a monitoring tool, you can calculate the **number of idle threads** by comparing the **current thread count** to the **number of busy threads**. The number of idle vs. busy threads provides a good measure for fine-tuning your server.
If your server has too many idle threads, it may not be managing the thread pool efficiently. If this is the case, you can lower the `minSpareThreads` value for your connector, which sets the minimum number of threads that should always be available in a pool (active or idle). Adjusting this value based on your application’s traffic will ensure there is an appropriate balance between busy and idle threads.

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
     maxThreads="<DESIRED_MAX_THREADS>"
     acceptCount="<DESIRED_ACCEPT_COUNT>"
     minSpareThreads="<DESIRED_MIN_SPARETHREADS>">
</Connector>
```



- If these parameters are set **too low** then the server will not have enough threads to manage the number of incoming requests, which could lead to **longer queues** and **increased request latency**.
- If the values are set too high or too low, then your server could receive an **influx of requests** it can’t adequately process, **maxing out** the **worker** threads and the request queue. 
  - This could cause requests in the queue to **time out** if they have to **wait longer** than the value set for the server’s `connectionTimeout`. 
  - A high value for `maxThreads` or `minSpareThreads` also increases your server’s startup time, and running a larger number of threads consumes more server resources.



![tomcat-error-jmx-attributes](https://raw.githubusercontent.com/chenzhengbc/md-images/develop/uPic/tomcat-error-jmx-attributes.png)

# Reference

1. [tomcat arthitecture and performance](https://www.datadoghq.com/blog/tomcat-architecture-and-performance/)



