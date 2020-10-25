# Blogs

**Blogs** is my record of the process of programming. JDK8, Dubbo-Java and Origin Blogs are included yet. 

## JDK8

This section is an analysis of the Java source code
(JDK8), includes collections (in java.util) and concurrency tools (in java.util.concurrent). The former covers the most commonly used data structures, such as List, Queue, and Map. (It is worth mentioning that HashMap in this package appears in almost every java project.) The latter is one of the secrets of Java's longevity (especially with the popularization of multi-core processors), which includes ThreadLocal(not in java.util.concurrent), Synchronizer, Concurrency Collections, and Thread Pool.

### Collections in java.util

| **Category** | **Content** |
| :- | :- |
| List, Stack, Queue | ArrayList, LinkedList, Stack, ArrayDeque, Priority |
| Set | HashSet, TreeSet |
| Map | HashMap, TreeMap, LinkedHashMap |

### Concurrency Tools in java.util.concurrent

| **Category** | **Content** |
| :- | :- |
| ThreadLocal | ThreadLocal |
| Synchronizer | AbstractQueuedSynchronizer, Reentrant, ReentrantReadWriteLock, CountDownLatch, CyclicBarrier, Phaser, Semaphore, Exchanger |
| Collections | *CAS*, CopyOnWriteArrayList, ConcurrentHashMap, ConcurrentSkipListMap, ArrayBlockingQueue, LinkedBlockingQueue, LinkedBlockingDeque, PriorityBlockingQueue, ConcurrentLinkedQueue, ConcurrentLinkedDeque, LinkedTransferQueue, SynchronousQueue, DelayQueue |
| Thread Pool | FutureTask, CompletableFuture, ThreadPoolExecutor, ScheduledThreadPoolExecutor |

*See more [details](https://github.com/Augustvic/Blogs/tree/master/JDK8).*

## Dubbo

My first pull request comes from Dubbo, which has been merged into the master branch soon. Then several ideas come to my mind and are submitted yet. It’s my great honor to be involved in the construction of this open-source project, which allows me to gain a lot.

Most of the following source code analysis is based on the official website and code comments, a few are original. All the blogs are written in Chinese, and for reference only.

* Contribution
* Structure and SPI
* Implementation of a remote process call
* Components of Netty and its application in Dubbo
* RPC extension
* Common model

*See more [details](https://github.com/Augustvic/Blogs/tree/master/Dubbo).*

## Blog

This section is for other blogs rather than source code analysis, mainly includes the following parts yet: 

* Java
* Database
* Distribution and Service
* Network
* Design Pattern

*See more [details](https://github.com/Augustvic/Blogs/tree/master/Blog).*
