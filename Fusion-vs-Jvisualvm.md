## Jvusialvm vs FusionReactor

## Table of content
1. [Introduction](#introduction)
2. [Java profiling](#java-profiling)
3. [FusionReactor vs Visualvm](#fusionreactor-vs-visualvm)
    1. [GUI vs Web](#gui-vs-web)
    2. [Default page](#default-page)
    3. [Main JVM metrics](#main-jvm-metrics)
        1. [GC](#gc)
        2. [Memory monitoring](#memory-monitoring)
        3. [JIT](#jit)
        4. [Threads](#threads)
        5. [JDBC](#jdbc)
        6. [System resources](#system-resources)
        7. [Plugins system](#plugins-system)
4. [Conclusion](#fusionreactor-right-for-you%3F)

## Introduction
A lot of new customers who have never tried to monitor their Java applications with FusionReactor might wonder what FusionReactor can do that VisualVM can't ,
If you have the same question in mind then welcome to a new article where we tried to compare both tools so you can decide which will work best for your use cases.

## Java profiling
Application profiling in general is a hard task. The usual purpose of profiling is to determine which sections of a program to optimize to increase its overall speed. Luckily , the awesome ecosystem of JVM makes it much easier for regular developers. 
But first you need to answer the simple question. What kind of monitoring do you need for your business? Do you want to know how Garbage Collector(GC) behaves in the pick overloads or the class name that eats all of the available heap space ? If you answer yes to all then VisualVM and FusionReactor are good starting points for you. 
The curious reader might ask , how do these tools work under the hood ? How does the FusionReactor collect the metrics of your running java process ? The answer is **Java agents**

### Java agents
Java agents are a special type of class which, by using the Java Instrumentation API, can intercept applications running on the JVM, modifying their bytecode. The dynamic nature of Java makes it really easy to add these interceptors on the fly or specify agent as a command line argument to your application and if you will take a closer look at FusionReactor setup guide you will see that it literally asks you to specify an agent
```
java  -javaagent:/home/strogiyotec/Java/fusion/fusionreactor.jar -agentpath:/libfrjvmti_x64.so -jar application.jar
```

## FusionReactor vs VisualVM
Before we start, let's have a brief introduction to VisualVM . According to Oracle, `jVisualVM is a visual tool integrating command line JDK tools and lightweight profiling capabilities. Designed for both development and production time use`. It's distributed as a part of jdk installed in the host machine.
Now let's walk through a step by step comparison of both technologies. 
### GUI vs Web
Both FusionReactor and Java VisualVM are graphical interfaces to view stats of remote Java applications in real time.
VisualVM is a Java-based desktop application, whereas FusionReactor offers a web interface. 
VisualVM also requires the client to have a JDK installed. Ideally, it should be easy to profile live applications in response to incident reports. FusionReactor offers a web interface so anyone with a web browser can profile an application.

What is the big deal here ? Our team in FusionReactor believes that fast response to production level problems is essential for all businesses in respect to their customers. In case when the dev team is not available , non technical people won't be able to(without proper training) install jdk , launch VisualVM , connect to the production environment and take a live snapshot of the issue. With FusionReactor it is just a matter of opening a new tab in the browser that everyone with an internet connection can do .

### Default page
To compare both tools, we have prepared a simple spring-boot project packaged as a single [fat jar](https://docs.spring.io/spring-boot/docs/current/reference/html/executable-jar.html). The application is running on port 8080 and has a single GET endpoint `/hello` which inserts a random row into the in memory H2 database. The source code is available [here](https://github.com/strogiyotec/fusion-reactor-jvisualvm-blog)(Check out the github repo on how to build and run the project). Now when the project is up and running let's open jvisualvm( typing `jvisualvm` in terminal should be enough if you have jdk installed).

![VisualVM first page](Jvisualvm-firstpage.png)

From this list choose your application by pid(in this case it's 70017)
Here is how the user interface in VisualVM will look for openjdk 11.
![](VisualVM-main-interface.png)

In the default **Overview** tab you can see
1. JVM flags that were passed to java process(in this case no flags)
2. The JVM version and vendor
3. Other system properties

What about FusionReactor ? First you need to run the jar file with a Java agent attached. The command will look like this
```
java  -javaagent:/fusionreactor.jar=address=8088 -agentpath:/libfrjvmti_x64.so -Dfrlicense=LICENSE_NUMBER -Dfradminpassword=COMPICATED_PASSWORD -jar fusion-reactor.jar
```
Once the app is deployed, open the admin page at the browser `http://localhost:8088` (the password will be the one specified by the `Dfradminpassword` parameter). 
This is the default interface 
![Fusion first image](Fusion-first-image.png)
First you can see that the interface has a lot more tabs compared with VisualVM. Next , the default page can be set by opening the tab you are interested in and pressing the **Set home page** button. In the screenshot above the Metrics tab is the default.

## Main JVM metrics
In this section we are going to compare how both tools deal with JVM metrics such as Garbage collection, memory monitoring , Threads activity and Just in time compilation(JIT)
### GC
Both VisualVM and FusionReactor support the insides of the garbage collector used by the application. The screen below shows how GC tab looks like in Jvisualvm
![VisualVM gc](Jvisual-gc.png)
You can see main GC statistics here such as 
1. Number of collections
2. Eden , Survivor , Old and Metaspace sizes. 

Statistics refresh rate can be adjusted with a dropdown in the top left corner.
To tune garbage collection settings for performance, we need to know which garbage collector the application is using. In this interface it's hard to tell which type of garbage collector it is.
 There are many GC types the JDK ships and defaults vary between versions and vendors. For our demo application, VisualVM shows a small green label called **G1 Evaluation Pause** suggesting that the Garbage First (G1) collector is being used.
Let's take a look at the FusionReactor interface for GC. First go to the **Resources** tab and open **Garbage collection** section.  
![Fusion gc](Fusion-gc.png)

With simpler screens it is a bit easier to tell which collector is used: The chart labels suggest that the G1 collector is being used. FusionReactor features many of the charts from VisualVM so new users coming from VisualVM won't have any trouble understanding the UI .In addition FusionReactor automatically records and persists garbage collection logs. With VisualVM you will have to collect these logs manually.

### Memory monitoring 
Next let's compare Memory monitoring starting with VisualVM
![jvisualvm-memory](jvisalvm-memory.png)

Heap usage is a good metric to start with, but I feel this is not enough to troubleshoot the actual cause of a memory related issue. 

How is FusionReactor different? 
First, it has several tabs for memory monitoring under the **Resources** tab. The default page is the **Memory overview** 
![Fusion memory](Fusion-memory.png)
It shows heap, non heap, code heap and compressed class space. As with garbage collection, all metrics are recorded and persisted for future viewing. Most java servers are running on top of servlet containers such as Tomcat or Jetty , if you need to see how many buffers are used to store HTTP requests then **Buffer Pools** tab will be your way to go(VisualVM doesn't support it natively you will have to install a third party plugin to see it , We come back to this later when we compare plugins system)

### JIT
JIT compilation can be a silent performance killer particularly when the compiler makes wrong assumptions about the code causing it to recompile hot methods again and again wasting CPU cycles. VisualVM doesn't offer any JIT related information. FusionReactor's **Resources** tab plots JIT compilation time. It can be filtered by specific time periods. 

![JIT](JIT.png)
For more JIT related metrics consider trying [JIT watch](https://github.com/AdoptOpenJDK/jitwatch) from AdoptOpenJDK


### Threads
When developing a high performance Java backend it's essential to understand how many threads will serve user requests and what these threads will actually do. If most of them are stacked in a blocking state there could be a huge I/O bottleneck. Both tools show all the threads in the application with their names and current states(This is shown in Threads tab in VisualVM and **Resources->Threads** in FusionReactor).
Here is the FusionReactor Threads page.
![Threads](Threads.png)
FusionReactor has a few benefits over VisualVM in terms of Threads monitoring.
1. You can filter threads by their states(This is useful for debugging I/O problems it can come in handy).
2. You can see formatted stack traces for each individual thread.
![Pretty Thread](PrettyThread.png)
3. You can stop any threads
>(**REMEMBER** stopping threads is considering as a dangerous action, For example if the stopped thread was holding a monitor it may cause a deadlock in your application)


## JDBC
In terms of web development apart from servlet container threads, developers are interested in database performance(using JDBC) to ensure a good user experience.
VisualVM has a nice JDBC profiling feature that shows all SQL queries sent to the database. Similarly, FusionReactor offers JDBC profiling with some additions. In the **JDBC** tab, apart from  all SQL queries, users can check the ratio of successful to unsuccessful queries and track sql exceptions. Also, for most SQL databases because of the way the delete and update queries work, it's essential to reduce the amount of time a single transaction takes. To monitor transaction time, FusionReactor has tabs to view the longest and slowest transactions respectively. 
![Fusion transactions](Fusion-transactions.png)

Java developers enjoy the abstraction that the JPA specification brings us.
Performing all database operations through an "object relational mapping" library(ORM) can speed up development, reduce mistakes, and decouple business logic from the particular data store.  An alternative is to manually write queries and map their results to native data structures, but this can be duplicative, labor intensive, and error prone. Therefore, it makes sense to abstract away the most common operations. However, without easy introspection into how the ORM maps methods to queries on a per-operation basis, how does one know what the network load will be like in practice? How does one identify low-hanging queries to optimize without seeing the SQL in code? Even with the SQL queries available, how does one measure which ones are actually bottlenecks in production scenarios?

To answer these questions FusionReactor offers the **Transaction by Memory tab** which shows the total allocated memory per query 
![](Fusion-transaction-size.png)

## System resources
Sometimes ,  the cause of performance issues originate or are the result of interactions outside the JVM. In these cases a reasonable first step is to check the resources of the machine running the Java process.
VisualVM unfortunately does not report metrics for the host machine. In the case of Linux hosts one would usually connect to the server over **ssh** and use command line tools like **top** and **ps** to view machine resources. FusionReactor serves this use case with its convenient **System Resources** tab.
It plots several useful machine metrics

## Plugins system
VisualVM supports third party plugins by default that you can install using the settings tab. Even Though FusionReactor doesn't support the plugin system at the time of writing, it does contain all the features that VisualVM plugins implement.

1. Network. How much data the host machine has sent over the wire
2. System memory 
3. CPU usage 
4. Disk usage. The read/write ratios over some period of time 
5. Processes. All the processes running on the host machine
![System metrics](SystemMetrics.png)
As with the other metrics, this data is persisted on our servers so you can respond to and understand incidents after they happen.

## FusionReactor right for you?
VisualVM is a great tool for a simple Java profiling.FusionReactor brings profiling to the next level to meet the needs of your organization's production applications. A web-based client allows anyone to see key, real-time metrics when it matters. From JVM and common host machine metrics, to live debugging and tools to harden your application's security(both of these features were not covered in this blog).
FusionReactor can help you reach your observability and live operations goals.
If you are wondering whether FusionReactor is the right observability solution for your project, please feel free to reach out for a personalized demo at [FusionReactor](https://www.fusion-reactor.com/)
