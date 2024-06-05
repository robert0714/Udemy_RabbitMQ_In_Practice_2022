# 8. Performance testing
## Performance testing tools
* jMeter
  * Plugin is available here:   
    https://github.com/jlavallee/JMeter-Rabbit-AMQP#build-dependencies
* Benchmark testing with PerfTest
  * Official RabbitMQ throughput testing tool:   
    https://github.com/rabbitmq/rabbitmq-perf-test
## RabbitMQ PerfTest
Download percompiled version of PerfTest    
https://github.com/rabbitmq/rabbitmq-perf-test/releases/
1. Run
``runjava.bat com.rabbitmq.perf.PerfTest --help``
or
``./runjava com.rabbitmq.perf.PerfTest --help``
### PerfTest basic usage
```
\bin\runjava.bat com.rabbitmq.perf.PerfTest --help
```
Or
```
\bin\runjava.bat com.rabbitmq.perf.PerfTest <test parametres> -h amqp://user:pass@<ip>:5672
```
### RabbitMQ performance testing
* 1 producer, 2 consumers, 1kB message size
  ```bash
  > runjava com.rabbitmq.perf.PerfTest --producers 1 --consumers 2 --queue "q.test-1" --size 1000 --autoack
   --time 20 --id "test 1" -h amqp://user:pass@<ip>:5672
  ```
* throughput vs latency
  ```bash
  > runjava com.rabbitmq.perf.PerfTest --producers 1 --consumers 1 --queue "q.test-1" --size 1000000 --autoack
   --time 20 --id "test 2" --framemax 5000

  > runjava com.rabbitmq.perf.PerfTest --producers 1 --consumers 1 --queue "q.test-1" --size 1000000 --autoack
   --time 20 --id "test 2" --framemax 2000000
  ```
* custom queue attributes
  ```bash
   > runjava com.rabbitmq.perf.PerfTest --time 20 --producers 1 --consumers 2 --queue "q.test-2" --size 1000
   --autoack --queue-args x-max-length=10,x-max-priority=5,x-dead-letter-exchange=ex.dlx-exchange-name
   --auto-delete false --id "test 3"
  ```
* Lazy queues test
  ```bash
   > runjava com.rabbitmq.perf.PerfTest --producers 50 --consumers 0 --queue "q.lazy-queue" --size 1000
     --autoack --id "test 1" -f persistent --auto-delete false --queue-args x-queue-mode=lazy --time 20
  
   > runjava com.rabbitmq.perf.PerfTest --producers 50 --consumers 0 --queue "q.not-lazy-queue" --size
     1000 --autoack --id "test 1" -f persistent --auto-delete false --time 20
  ```
* Ramp-Up test
  ```bash
    > for i in {1..10}; do ./runjava com.rabbitmq.perf.PerfTest \
     --time 240 --producers 1 --consumers 1 \
     --queue "q.test-2" --size 1000 --autoack \
     --rate 1000 \
     --id "test 5" -h amqp://guest:guest@localhost:5672 & sleep 5; done
 
  ```
* Throughput in function of number of producers
  ```bash
    > runjava com.rabbitmq.perf.PerfTest --time 20 --queue-pattern 'q.perf-test-%d'
    --queue-pattern-from 1 --queue-pattern-to 10 --producers 1 --producer-channel-count 10
    --consumers 0 --size 100 --queue-args x-queue-mode=lazy --flag persistent

    > runjava com.rabbitmq.perf.PerfTest --time 20 --queue-pattern 'q.perf-test-%d'
    --queue-pattern-from 1 --queue-pattern-to 10 --producers 10 --consumers 0
    --size 100 --queue-args x-queue-mode=lazy --flag persistent 
  ```
*  **Simulate IoT workloads without requiring too many resources, especially threads**
   Share sockets and threads when not used. NIO stands for non-blocking I/O operations.
    | parameters                 |                                       |
    |----------------------------|---------------------------------------|
    | --nio-threads 10             | non-blocking IO enabled with pool of 0|
    | --producer-scheduler-threads 10  | by default on thread is used to simulate 50 producers. I declared 2000 producers, so **instead of 40, only 10 threads will be used**. Publishers publish 1 message/s, so we can limit it to 10 to save machine resources              |
    | --consumers-thread-pools 10  | by default pool of threads is equal to the number of consumers. I declared 2000 consumers, so **instead of 2000 threads, only 10 will be used** |
    | --heartbeat-sender-threads 10 | **separate thread is used** for every producer and every consumer **to send heartbeats**. Set limit to reasonable value when using lot of producers and consumers and avoid java.lang.OutOfMemoryError: unable to create new native thread |
    | --publishing-interval 1     | publish message every second    |
    | --producer-random-start-delay 60 | producers randomly start between 1s and 60s |

   ```bash
   > runjava com.rabbitmq.perf.PerfTest --time 120 --queue-pattern 'perf-test-%05d'
   --queue-pattern-from 1 --queue-pattern-to 2000 --producers 2000 --consumers 0 --nio-threads 10
   --producer-scheduler-threads 10 --consumers-thread-pools 10 --publishing-interval 1 --size 512
   --heartbeat-sender-threads 10 --flag persistent --producer-random-start-delay 60
   ```
### PerfTest - Summary
1. **Stability**   
Cluster to survive in a function of growing load (Ramp-Up test), stable load (Flat Line test) or
mixed (Flat Line with occasional high peaks)
2. **Maximum throughput and minimum latency**   
Realize limits of your cluster, define SLA, be prepared for scaling-out nodes when number of
published messages is growing
3. **Best optimal architecture**   
Which plugin, how many exchanges, what type of exchanges, what type of the the Queue
(Classic, Quorum, Stream), how many mirrors etc.
4. **Best optimal publication and consumption parameters**   
Frame size, batching, prefetch, transactions, publisher confirms etc.

## HTML Performance tools
Use HTML Performance Tools to visualize results:   
https://github.com/rabbitmq/rabbitmq-perf-test/blob/master/html/README.md


## HTML Performance tools
* **Simple**  
  4 producers, 2 consumers, 30s test: how time affect on message rate per second and latency
  ```bash
   > rabbitmq-perf-test-2.1.2\bin\runjava.bat com.rabbitmq.perf.PerfTestMulti various-spec.js
     various-result.js
  ```
* **Batch test**
  - 1 producer, 1 consumer, 30s no ACKs: how time affect on message rate per second and latency
  - how number of producers affect message rate per second,
  - rate message sizes: how message size affects the message rate per second,
  - rate attempted vs latency: compare the sending rate of messages vs. the latency,
  ```bash
   > rabbitmq-perf-test-2.1.2\bin\runjava.bat com.rabbitmq.perf.PerfTestMulti publish-consume-spec.js
    publish-consume-result.js
  ```

