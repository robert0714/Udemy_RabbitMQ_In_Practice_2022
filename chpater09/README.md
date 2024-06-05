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
