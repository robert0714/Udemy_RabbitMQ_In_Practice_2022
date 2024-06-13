# 9. Monitoring Hands On
https://github.com/bigdotsoftware/rabbitmq-tools

## Introduction
### Monitoring - why?
* **Prevent not cure**  
Monitor to to avoid infrastructure issues before they appear and cause serious consequences. Respond to
issues proactively, **preventing loss of time and money**. Minimize downtime and increase operational
efficiency.
* **Detect anomalies / understand the characteristics of your data**  
Prepare for high spikes and unusual load by understanding the characteristics of your data (Is growing on
unexpected load a DDOS attack or regular behaviour?)
* **Autoscaling**   
Scale-out your RabbitMQ nodes when facing high data ingest due to planned and unplanned traffic spikes.
Scale-in when load is coming back to the typical value.
* **Notify right people (alerting)**   
When facing an outage - notify responsible people ASAP
* **Find the root cause**   
Drill down into specific infrastructure component - precisely determine where a problem originates

### Monitoring - how?
* **RabbitMQ RESTful API**   
Requires RabbitMQ management plugin installed. By default:
``http://localhost:15672/api/``
Script is a part of the management plugin:
  ```bash
  rabbitmqadmin --help
  ```
* **Command line tools**
They communicate using RabbitMQ distribution port (25672 by default). Root
privileges needed.
  ```bash
  rabbitmqctl --help
  rabbitmq-diagnostics --help
  ```
  > Output can be formatted: **json**, **csv**, **erlang**,**pretty_table**, **table**

### Monitoring - what?
1. **Cluster metrics**   
typical cluster metrics, like number of nodes,
alarms, partitions and total rates and counts to
understand the typical work environment
of our cluster
   > Management plugins to help: rabbitmq-plugins enable **rabbitmq_top**   
   > Set-up memory and disk alarms. Producers will be blocked once threshold is met
2. **Each node metrics**  
uptime, CPU, memory, free disk space,
partitions, high churn rates
3. **Queues size, age and resources**   
Some queues are more important than others, so depends on particular resources we will
choose different set of metrics. Most common scenario is to monitor rates on queues and
exchanges.

## Monitoring - collecting metrics
### Cluster metrics
* cmd
  ```bash
  > rabbitmqctl cluster_status
  > rabbitmqctl cluster_status --formatter json
  ```
* api
  ```bash
  > curl -XGET http://localhost:15672/api/overview
  ```
### Cluster metrics summary

  |                                                                           | Command line tools                          | RESTful API    | Monitoring |
  |---------------------------------------------------------------------------|---------------------------------------------|----------------|------------|
  | Cluster name                                                              | y                                           | y              |            |
  | Disk Nodes                                                                | y                                           | Use /api/nodes |            |
  | Running Nodes                                                             | y                                           | Use /api/nodes | ✔          |
  | Versions                                                                  | y                                           | y              |            |
  | Alarms                                                                    | y                                           | Use /api/nodes | ✔          |
  | Network partitions                                                        | y                                           | Use /api/nodes | ✔          |
  | Listeners                                                                 | y                                           | y              |            |
  | Feature flags                                                             | y                                           | y              |            |
  | Exchange types                                                            | n/a                                         | y              |            |
  | Message statistics                                                        | n/a - feature of RabbitMQ<br/> management plugin | y              | ✔          |
  | Churn rates                                                               | n/a - feature of RabbitMQ<br/> management plugin | y              | ✔          |
  | Totals<br/> (number of all<br/> messages, queues,<br/> exchanges, connections<br/>, channels) | Use other parameters, like:<br/>list_queues, list_exchanges <br/> etc.|y|           |


### Node metrics
* cmd
  ```bash
  > rabbitmqctl status
  > rabbitmqctl status --formatter json
  > rabbitmqctl report
  > rabbitmq-diagnostics memory_breakdown
  ```
* api
  ```bash 
  > curl -XGET http://localhost:15672/api/nodes
  > curl -XGET http://localhost:15672/api/nodes/<nodename>
  > curl -XGET http://localhost:15672/api/nodes/<nodename>?memory=true&binary=true
  ```
### Node metrics summary

  |                                                                     | Command line tools                          | RESTful API    | Monitoring |
  |---------------------------------------------------------------------|---------------------------------------------|----------------|------------|
  | Uptime                                                              | y                                           | y              | ✔          |
  | Versions                                                            | y                                           | y              |            | 
  | Alarms                                                              | y                                           | y              | ✔          |
  | Erlang processes                                                    | y                                           | y              | ✔?          |
  | File and socket descriptors                                         | y                                           | y              | ✔?         |
  | Free disk space                                                     | y                                           | y              | ✔?         |
  | Total allocated memory                                              | y                                           | y              | ✔          |
  | Memory breakdown                                                    | n - use cluster_status                      | y              | ✔?          |
  | Partitions                                                          | n/a - feature of RabbitMQ<br/> management plugin | y         | ✔          |
  | Churn rates (i.e. connections,<br/>channels, queues, exchanges,<br/>vhosts)|n/a - feature of RabbitMQ<br/> management plugin |  y  | ✔          |


### Node metrics  Channels monitoring
  * **Channel leaks**   
    Application (publisher or consumer) opens channels without closing them
    ```bash
    > curl -XGET 'http://127.0.0.1:15672/api/nodes/'
    > rabbitmqctl list_connections name channels -q
    ```

  * **High channel churn**
    Both: Rate of newly opened channels rate of closed channels are consistently high
    ```bash     
     > curl -XGET 'http://127.0.0.1:15672/api/nodes/'
    ```
  * To avoid “High channel churn” while interacting with i.e. Bash, we need to introduce an additional layer into our architecture, like RESTful API service.
    ```bash     
     > rabbitmqctl list_connections name channels -q
    ```

### Resources metrics
* cmd
  ```bash
  > rabbitmqctl list_exchanges
  > rabbitmqctl help list_exchanges
  > rabbitmqctl list_exchanges --formatter json

  > rabbitmqctl list_queues
  > rabbitmqctl help list_queues
  > rabbitmqctl list_queues --formatter json

  > rabbitmqctl list_vhosts
  > rabbitmqctl list_policies
  > rabbitmqctl list_users
  ```
* api
  ```bash 
  > curl -XGET http://localhost:15672/api/exchanges
  > curl -XGET http://localhost:15672/api/queues
  > curl -XGET http://localhost:15672/api/vhosts
  ```

### Resources metrics - Summary

  |                                                                     | Command line tools                          | RESTful API    | Monitoring |
  |---------------------------------------------------------------------|---------------------------------------------|----------------|------------|
  | Queue rates (created, deleted,<br/>publish, consume)                |n/a - feature of RabbitMQ<br/> management plugin| y              | ✔          |
  | Queue bytes (publish,<br/> consume)                                 |n/a - feature of RabbitMQ<br/> management plugin| y              | ✔          | 
  | Queue size                                                          | y                                           | y              | ✔          |
  | Exchanges (publish rates and <br/> number of bytes)                 |n/a - feature of RabbitMQ<br/> management plugin| y              | ✔          |
  | Channels rate (created, <br/> deleted, publish, consume)            |n/a - feature of RabbitMQ<br/> management plugin| y              | ✔          |
  | Channels (number of bytes <br/> published and delivered)            |n/a - feature of RabbitMQ<br/> management plugin| y              | ✔          |
  | Users (number of connected)                                         | y                                           | y              | ✔          |
  | Consumers                                                           | y                                           | y              |             |
  | Connections (send, recv bytes)                                      |n/a - feature of RabbitMQ<br/> management plugin | y         | ✔          |
  | Bindings                                                            | y                                           |  y             |          |
  | VHosts (send bytes, recv <br/>bytes, rates)                         |n/a - feature of RabbitMQ<br/> management plugin |  y         | ✔          |  

## Monitoring - memory model
### Queues - memory
* **Queues always keep part of messages in memory**   
RabbitMQ deliver messages to consumers as fast as possible
* **Queue is an Erlang process**   
Has its own heap (security & reliability)
* **Body is stored separately in a separate memory**   
Body of the message is stored in “Binaries”
* **Service booting**   
When RabbitMQ starts, up to 16384 messages
smaller than 4k are loaded into memory

### Memory test - result
300 000 messages * 1kB each = 300 MB

 |  Test #1  | **Start**   | **End**    |
 |-----------|-------------|------------| 
 |  Binaries |    0        |   322 MB   | 
 |Memory Queue|  0        |   194 MB   | 
 |Memory RSS|  256 MB        |   814 MB (↥558 MB)   |  
 

 |  Test #2  | **Start**   | **End**    |
 |-----------|-------------|------------| 
 |  Binaries |    0        |   321 MB   | 
 |Memory Queue|  0        |   584 MB   | 
 |Memory RSS|  254 MB        |   1.4 GB (↥1.1 GB)   |  
 
> We don’t consume 3x more memory, but we store 3x more messages


### Memory breakdown

```bash
> rabbitmq-diagnostics memory_breakdown
> curl -XGET 'http://127.0.0.1:15672/api/nodes/rabbit1@localhost/memory'
> curl -XGET 'http://127.0.0.1:15672/api/nodes/rabbit1@localhost?memory=true&binary=true'
```

### Memory calculation strategy
* **Legacy (erlang)**   
Oldest one and fairly inaccurate strategy - underreport the real memory usage.
* **Allocated**  
Available from version 3.6.11 (August 2017). It queries Erlang memory allocator for
details. This strategy is used by default on Windows.
* **RSS**   
Available from version 3.6.11 (August 2017). OS-specific approach - it queries the kernel
to find RSS (Resident Set Size) value of the process. This strategy is most precise and used
by default on unixes

> How memory is calculated:
> vm_memory_calculation_strategy = legacy/rss/allocated

> Currently RabbitMQ calculates memory usage using both
> strategies at the same time, so you can compare the
> difference. Selected strategy tells RabbitMQ which value
> should be used to determine if the memory usage reaches
> the watermark or paging to disk is required.

### Queue Metrics monitoring
* **Queue size**   
How many messages wait to be consumed.
* **Maximum messages age**  
Similarly to IteratorAge in AWS Kinesis - number of ms message spent in the queue. There is no out-of-the-box solution, measure it manually.
* **Incoming / Outgoing bytes**  
Throughput in bytes
* **Incoming / Outgoing messages**  
Throughput in number of messages

Messages age (Iterator age) - this need to be implemented manually. There are two suggested approaches
* **Consumer oriented**  
Producer to store publication timestamp in message header. Consumer to measure time and emit metric. Known issue: no consuming = no metrics.
* **Producer oriented**  
Producer to emit metric about message publication + store publication timestamp in message header. Consumer to close the metric.

### Node statistics monitoring
* **Connections and Channels**   
Number of connections and channels, channel leaks and “High channel churn”
* **Memory usage**   
Memory breakdown is useful to decide on a possible optimizations
* **Active Partitions**  
Expected result is : No active partitions
* **Active Alarms**

### Monitoring - Other
Use rabbitmq management plugin, RESTful API , rabbitmqctl or rabbitmq-diagnostics command line tools
* Use plugin
  ```bash
  > rabbitmq-diagnostics memory_breakdown
  > rabbitmq-diagnostics check_if_node_is_quorum_critical
  ```
* Use rabbitmqctl
  ```bash
  > rabbitmqctl status
  > rabbitmqctl list_queues
  ```
* RESTful API
  ```bash
  curl -s -XGET http://localhost:15672/api/overview
  curl -s -XGET http://localhost:15672/api/nodes
  curl -s -XGET http://localhost:15672/api/nodes/<name>
  curl -s -XGET http://localhost:15672/api/connections
  curl -s -XGET http://localhost:15672/api/channels
  curl -s -XGET http://localhost:15672/api/channels/<name>
  curl -s -XGET http://localhost:15672/api/exchanges
  curl -s -XGET http://localhost:15672/api/exchanges/<name>
  curl -s -XGET http://localhost:15672/api/queues
  curl -s -XGET http://localhost:15672/api/vhosts/
  ```

### Monitoring - Other
Tracer is very simple AMQP 0-9-1 protocol analyzer. The easiest way to play with it is to locate
Tracer between Producer and RabbitMQ.
1. Stop RabbitMQ
2. Start RabbitMQ on different port, i.e 5678 by setting:   
   * in windows
     ```bash
     set RABBITMQ_NODE_PORT=5678
     ```
   * in linux
     ```bash
     export RABBITMQ_NODE_PORT=5678
     ```     
3. Start RabbitMQ

4. Run trace
   ```bash
   > runtracer 5672 localhost 5678
   ```

### Monitoring - Tracer  
```mermaid
flowchart LR
    P((Producers))
    T1[/Tracer/]
    T2[/Tracer/]
    Q1[[RabbitMQ]]
    C1((Consumer))

    P -- :5672 --> T1
    T1 -- :5678 --> Q1
    Q1 -- :5678 --> T2
    T2 -- :5672 --> C1

    class P mermaid-producer
    class T1 mermaid-tracer
    class T2 mermaid-tracer
    class Q1 mermaid-queue
    class C1 mermaid-consumer
```
## Monitoring - Alarms  
### RabbitMQ Alarms
RabbitMQ set and respect thresholds. When they are reached, RabbitMQ raises and alarm
and block connections that publish messages. Alarms can be raised:
* When memory goes above the safe limit
* When free disk space drops below the safe limit
* When number of file and socket descriptors are too high
* When number of Erlang processes is too high

In all scenarios RabbitMQ temporarily blocks connections. Connection for heartbeat monitoring is also disabled.

There is additional “Flow Control” feature related to connections which publish or consume too fast.

* **File descriptors**  
Managed by operating system. On Unix use **“ulimit -n”**, on Windows set **ERL_MAX_PORTS** environment variable

* **Socket descriptors**  
Subset of file descriptors initially set as **80%-90% of file descriptors limit**. RabbitMQ gives priority to the network operations - with growing load more descriptors are being used by sockets and less are available for
files causing slower disk operations

* **Erlang processes**
One million by default. Use **RABBITMQ_MAX_NUMBER_OF_PROCESSES** or **RABBITMQ_SERVER_ERL_ARGS** environment variables. Increasing is not a good practice - scale out your cluster.

* **Memory**  
40% RAM by default. Use **vm_memory_high_watermark** in rabbitmq.conf file.

* **Free disk space**  
50MB by default. Use **disk_free_limit** in rabbitmq.conf file.

### Alarms - Summary
* **Never too high, never too low**  
Take into account type, size and volume of data which is processed by your cluster
as well hardware limits
* **Don't increase erlang processes**  
Default value of 2^20 = 1048576 is already very high. Scale-out your cluster by adding
more nodes (IoT workload)
* **Monitor alarms**  
Better prevent than cure

### RabbitMQ Memory Alarms
As default RabbitMQ memory threshold is set to 40% of available RAM. More memory can 
be allocated, just by passing 40% RabbitMQ will start throttle (block connections).

vm_memory_high_watermark.relative = 0.4
```bash
> rabbitmqctl set_vm_memory_high_watermark 0.6
> rabbitmqctl set_vm_memory_high_watermark absolute "4G
```

### RabbitMQ Disk Alarms
As default RabbitMQ disk threshold is set to 50MB. When free disk space drops below, an
alarm will be triggered and RabbitMQ will block connections.

disk_free_limit.absolute = 1GB
```bash
> rabbitmqctl set_disk_free_limit 1GB
```

### RabbitMQ Flow Control
There is additional “Flow Control” feature related to connections which publish or consume too fast. RabbitMQ uses “Credit Flow Control” algorithm. It’s not configurable, exchanges, queues, connections can be in “flow” state.