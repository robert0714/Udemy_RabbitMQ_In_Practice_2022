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
* summary

