# 8. Performance testing
## Performance testing tools
* jMeter
  * Plugin is available here:
https://github.com/jlavallee/JMeter-Rabbit-AMQP#build-dependencies
* Benchmark testing with PerfTest
  * Official RabbitMQ throughput testing tool:  https://github.com/rabbitmq/rabbitmq-perf-test
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
  ```bash
   > runjava com.rabbitmq.perf.PerfTest --time 120 --queue-pattern 'perf-test-%05d'
   --queue-pattern-from 1 --queue-pattern-to 2000 --producers 2000 --consumers 0 --nio-threads 10
   --producer-scheduler-threads 10 --consumers-thread-pools 10 --publishing-interval 1 --size 512
   --heartbeat-sender-threads 10 --flag persistent --producer-random-start-delay 60
  ```
