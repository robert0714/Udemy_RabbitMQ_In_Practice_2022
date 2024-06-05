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
