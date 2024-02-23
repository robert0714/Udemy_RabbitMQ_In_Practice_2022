# 4. Getting starting with RabbitMQ
## Prerequisites
* Hardware
  * Laptop or Virtual machine with Windows 10
* Software
  * Text editor, like Notepad++ or similar
  * Web browser, like Chrome or similar
  * (optionally) InteliJ to run Java code examples
* Skills
  * Using command line tools
  * Basic Windows administration knowledge
## Basic configuration and Installation Hands On
### Installation
For better understanding let’s follow manual installation steps
#### Tradition
1. Download RabbitMQ
   ```bash
   wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.12.13/rabbitmq-server-windows-3.12.13.zip
   choco install -y rabbitmq 
   ```
1. Download Erlang/OTP
   ```bash
   wget https://github.com/erlang/otp/releases/download/OTP-26.2.2/otp_win64_26.2.2.exe
   ```
1. Run the Rabbit!
   ```bash
   > cd %RABBITMQ%\rabbitmq_server-3.12.13\sbin
   > set ERLANG_HOME=c:\Program Files\erl-25.3.2.9
   > rabbitmq-server.bat
   
   ##  ##      RabbitMQ 3.12.13
   ##  ##
   ##########  Copyright (c) 2007-2024 Broadcom Inc and/or its subsidiaries
   ######  ##
   ##########  Licensed under the MPL 2.0. Website: https://rabbitmq.com

   Erlang:      25.3.2.9 [jit]
   TLS Library: OpenSSL - OpenSSL 3.1.5 30 Jan 2024
   Release series support status: supported

   Doc guides:  https://rabbitmq.com/documentation.html
   Support:     https://rabbitmq.com/contact.html
   Tutorials:   https://rabbitmq.com/getstarted.html
   Monitoring:  https://rabbitmq.com/monitoring.html

   Logs: <stdout>

   Config file(s): /etc/rabbitmq/conf.d/10-defaults.conf

   Starting broker...2024-02-22 17:56:14.041819+08:00 [info] <0.230.0>
   ```
#### Podman machine
* Install windows wsl2
  ```powershell
  wsl --install --no-distribution
  ```
* Install podman-cli
  ```powershell
  choco install -y podman-cli
  ```
* Start podman machine
  ```powershell
  podman machine init
  podman machine start
  ```
* Use docker-compose.yml (https://www.composerize.com/)
  ```yaml
  version: '3' 
  services:
    rabbitmq:
      image: rabbitmq:3.12.13-management
      container_name: rabbitmq
      restart: always
      mem_limit: 350m
      environment:
        - TZ=Asia/Taipei
      network_mode:  "host"  
      #ports:
      #  - 5672:5672
      #  - 15672:15672
      healthcheck:
        test: ["CMD", "rabbitmqctl", "status"]
        interval: 20s
        timeout: 5s
        retries: 10
  ```
* Use the below cli (https://www.decomposerize.com/)
  ```bash
  docker run --net host --name rabbitmq --restart always -e TZ=Asia/Taipei --health-cmd CMD,rabbitmqctl,status --health-interval 20s --health-retries 10 --health-timeout 5s -m 350m rabbitmq:3.12.13-management
  ```
#### Anthos sandbox
* Go to : https://ide.cloud.google.com/?boost=true
## Installation - plugins
RabbitMQ supports plugins. Plugins extend core broker functionality in a many ways
* more protocols
* monitoring
* additional AMQP 0-9-1 exchange types
* node federations
* and many many more

Default RabbitMQ distribution includes lot of plugins disabled by default.

Plugins can be enabled and disabled when node is stopped as well at runtime using command line tools.

## Installation - plugins (management)
Stop the RabbitMQ, then install web management plugin (alternatively open new console, set environment variables and enable plugin using the same command without stopping the node )
* more protocols
* monitoring
* additional AMQP 0-9-1 exchange types
* node federations
* and many many more

Default RabbitMQ distribution includes lot of plugins disabled by default.

Plugins can be enabled and disabled when node is stopped as well at runtime using command line tools.https://blog.rabbitmq.com/docs/plugins#basics

>> Alternatively install rabbitmq_top
   ```bash
   > cd %RABBITMQ%\rabbitmq_server-3.12.13\sbin
   > set ERLANG_HOME=c:\Program Files\erl-25.3.2.9
   > rabbitmq-plugins.bat enable rabbitmq_management

   ##  ##      RabbitMQ 3.12.13
   ##  ##
   ##########  Copyright (c) 2007-2024 Broadcom Inc and/or its subsidiaries
   ######  ##
   ##########  Licensed under the MPL 2.0. Website: https://rabbitmq.com

   Erlang:      25.3.2.9 [jit]
   TLS Library: OpenSSL - OpenSSL 3.1.5 30 Jan 2024
   Release series support status: supported

   Doc guides:  https://rabbitmq.com/documentation.html
   Support:     https://rabbitmq.com/contact.html
   Tutorials:   https://rabbitmq.com/getstarted.html
   Monitoring:  https://rabbitmq.com/monitoring.html

   Logs: <stdout>

   Config file(s): /etc/rabbitmq/conf.d/10-defaults.conf

   Starting broker...2024-02-22 17:56:14.041819+08:00 [info] <0.230.0>
   ```

## Configuration
RabbitMQ provides three general ways to customise the node:
* **environment variables** - define ports, file locations and flags (taken from the shell, or set in the
environment configuration file,rabbitmq-env.conf/rabbitmq-env-conf.bat)
* **configuration file** - defines server component settings for permissions, limits and clusters, and also plugin
settings.
* **runtime parameters and policies** - defines cluster-wide settings which can change at run time
* windows samples
   ```cmd
   cd c:\RabbitMQ\rabbitmq-server-windows-3.12.13_node1\sbin
   set HOMEDRIVE=C:
   set HOMEPATH=\Users\%USERNAME%\
   set ERLANG_HOME=c:\Program Files\erl-25.3.2.9
   set RABBITMQ_NODE_PORT=5672
   set RABBITMQ_DIST_PORT=25672
   set RABBITMQ_NODENAME=rabbit1@localhost
   set RABBITMQ_MNESIA_BASE=C:\data\rabbit1
   set RABBITMQ_MNESIA_DIR=C:\data\rabbit1\data
   set RABBITMQ_LOG_BASE=C:\data\rabbit1\logs
   REM Change rabbit1.conf; management.tcp.port = 15672
   REM The Erlang runtime automatically appends the .conf extension to the value of this variable.
   set RABBITMQ_CONFIG_FILE=C:\Users\Me\AppData\Roaming\RabbitMQ\rabbitmq1
   set RABBITMQ_ENABLED_PLUGINS_FILE=C:\data\rabbit1\enabled_plugins
   rabbitmq-server.bat
   ```
> Erlang is a general-purpose programming language and runtime environment

Most important configuration variables:
* **RABBITMQ_NODE_PORT** (default 5672) - used by AMQP 0-9-1 and 1.0 clients (without TLS)
* **RABBITMQ_DIST_PORT** (default 20000 + RABBITMQ_NODE_PORT) - used by Erlang distribution for inter-node and CLI tools communication,
* **RABBITMQ_NODENAME** (Unix default: rabbit@$HOSTNAME; Windows default: rabbit@%COMPUTERNAME%) - unique node name
* **RABBITMQ_MNESIA_BASE** - RabbitMQ server's node database, message store and cluster state files, one for each node
* **RABBITMQ_MNESIA_DIR** - This is **RABBITMQ_MNESIA_BASE** subfolder. Includes a schema database, message stores, cluster member information and other persistent node state
* **RABBITMQ_LOG_BASE** - just logs
* **RABBITMQ_CONFIG_FILE** - points rabbitmq.config file (note the “.config” extension is added)
* **RABBITMQ_ENABLED_PLUGINS_FILE** - file to track enabled plugins
### The New and Old Config File Formats
https://blog.rabbitmq.com/docs/configure#config-file-formats
Compare this examplary ``rabbitmq.conf`` file:  
```conf
# A new style format snippet. This format is used by rabbitmq.conf files.
ssl_options.cacertfile           = /path/to/ca_certificate.pem
ssl_options.certfile             = /path/to/server_certificate.pem
ssl_options.keyfile              = /path/to/server_key.pem
ssl_options.verify               = verify_peer
ssl_options.fail_if_no_peer_cert = true
```
to
```json
%% A classic format snippet, now used by advanced.config files.
[
  {rabbit, [{ssl_options, [{cacertfile,           "/path/to/ca_certificate.pem"},
                           {certfile,             "/path/to/server_certificate.pem"},
                           {keyfile,              "/path/to/server_key.pem"},
                           {verify,               verify_peer},
                           {fail_if_no_peer_cert, true}]}]}
].
```
Most important config file entries
* **cluster_name** (default “”) - used for automatic clustering
* **listeners.tcp.default** - same as **RABBITMQ_NODE_PORT**
* **heartbeat** (default 60 seconds) - after 60s connection should be considered unreachable by RabbitMQ and client libraries. Server suggest this value to client libraries while establishing TCP connection at AMQP protocol level. Clients might not follow server’s suggestion,
* **frame_max** - (default 131072) - maximum permissible size of a frame (in bytes) to negotiate with clients (AMQP protocol level). Larger value improves throughput, smaller value improves latency (not related with  max message size which is **2GB**)
* **channel_max** - (default 0 - unlimited) - number of channels to negotiate with clients. Using more channels increases broker’s memory footprint,
* **management.tcp.port** (default 15672) - the web-management and RESTful service
## Configuration file Hands On
* https://github.com/rabbitmq/rabbitmq-server/blob/master/deps/rabbit/docs/rabbitmq.conf.example   
## Web Admin Overview and Default User Password
### Tab: Connections
* **Connection**: TCP/IP connection between the client and the broker
* **Channel** : Virtual connection inside the physical TCP/IP connection (single TCP/IP connection with many channels vs many TCP/IP connections)

Let’s assume scenario:
* You don’t like web management UI
* You need to integrate external systems with RabbitMQ
* You need to create your own monitoring UI for RabbitMQ

Use RabbitMQ RESTful API instead!

## RabbitMQ RESTful API.
Web Admin is coming with RESTful service: ``http://localhost:15672/api/nodes``
   ```bash
   > cd %RABBITMQ%\rabbitmq_server-3.12.13\sbin
   > set ERLANG_HOME=c:\Program Files\erl-25.3.2.9
   > rabbitmq-plugins.bat enable rabbitmq_management
   ```
Make sure the following ports are open:
* **4369**: epmd, a peer discovery service used by RabbitMQ nodes and CLI tools
* **5672, 5671**: used by AMQP 0-9-1 and 1.0 clients without and with TLS
* **25672**: used by Erlang distribution for inter-node and CLI tools communication and is allocated from a dynamic range (limited to a single port by default, computed as AMQP port + 20000).
* **15672**: HTTP API clients and rabbitmqadmin (only if the management plugin is enabled) 

Other protocols:
* **61613, 61614**: STOMP clients without and with TLS (only if the STOMP plugin is enabled)
* **1883, 8883**: MQTT clients without and with TLS, (only if the MQTT plugin is enabled)
* **15674**: STOMP-over-WebSockets clients (only if the Web STOMP plugin is enabled)
* **15675**: MQTT-over-WebSockets clients (only if the Web MQTT plugin is enabled)  
## Core concepts
* **Producer** emits messages to exchange
* **Consumer** receives messages from the queue
* **Queue** keeps/stores messages
* **Exchange**
    * **Binding** connects an exchange with a queue using **binding key**
    * Exchange compares **routing key** with binding key (on-to-one or using pattern)
> From the day one take care about naming conventions!
>> Producer never send messages directly to the queue
    
## Exchanges, Queues, Bindings, Routing Keys
### Exchange
* Exchange type determines distribution model
* **Exchange types: nameless (empty string - default one), fanout,direct, topic , headers**
* Nameless exchange (aka “default exchange”, aka “AMQP default”)
    * Special one created by RabbitMQ
    * Compares routing key with queue name
    * Allows sending messages “directly” to the queue (from the publisher perspective exchange is transparent)
         ```bash
         > rabbitmqctl list_exchanges        
         ```
* **fanout**
Simply routes a received message to all queues that are bound to it. Ignores routing key
* **direct**
Routes a received message to the queue that match “rounting-key”. Nameless exchange is a “direct” exchange.
* **topic**
Routes a received message to queues, where binding key (defined as a pattern) matches to the
routing key. Example, binding-key: “*.logs.error”, routing-key: “aplication1.logs.error”
* **headers**
Same as topic, but binding-key is compared against “any” or “all” message headers (header
x-match determines the behavior)     
### Queue concept
For resilience and availability, queue can be Federated or Mirrored.
> Queue max size is set by policies
* **Located on single node where it was declared and referenced by unique name**
  * In contrary to Exchanges and Bindings, which exist on all nodes. Name can be provided or auto-generated by RabbitMQ. Both: Producer and Consumer **can create a queue**.
* **Performance**
  * 1 queue = 1 Erlang process
  * When node starts, up to **16384** messages are loaded into RAM from the queue
* **Queue is ordered collection of messages**
  * Messages are published and consumed in the FIFO manner (except prioritized queues)
* **Queue has many properties**
  * Depends form the use case we can set queue behavior
* **Queue can be federated or mirrored**
  * To increase reliability and availability
* **Internal queues amq.**
  * Queues prefixed by “amq.” are used for
  * RabbitMQ internal purposes only

```bash
> rabbitmq-diagnostics memory_breakdown
> rabbitmqctl status
> rabbitmqctl list_queues    
```
### Queue properties
Among others RabbitMQ implements AMQP protocol. So, attributes (properties) and queue
behavior can be changed in many ways to support generic RabbitMQ architecture.
* By common Queue definition
  * Name, Durable, Auto-Delete, Exclusive
    ```java
    channel.queueDeclare(QUEUE_NAME, /*durable*/true, /*exclusive*/false, /*autoDelete*/false, /*arguments*/args);
    ```   
* By protocol specific settings
  * Priority etc.
    ```java
    Map<String, Object> args = new HashMap<String, Object>();
    args.put("x-max-priority", 10);
    channel.queueDeclare(..... /*arguments*/args);
    ```
* By policies
  * TTL, Federation etc. (web management, REST API, command line tool)
    ```bash
    > rabbitmqctl set_policy TTL ".*" "{""message-ttl"":60000}" --apply-to queues
    ```
### Queue properties (most popular)
* Name
  * Name can be provided or auto-generated by RabbitMQ
* Durable or not
  * Not durable queues won’t survive broker restart
* Auto Delete feature
  * Queue deletes itself when all consumers disconnect
* Classic or Quorum
  * Quorum and Mirroring (policy) increases availability
* Exclusive
  * Used by only one connection and the queue will be deleted when that connection closes
* Priority
  * Additional CPU cost and increased latency; no guarantee of exact order (just strong suggestion)
* Expiration time (TTL)
  * Both messages (expiration property) and queues (x-message-ttl) can have TTL (policy settings); minimum value from both is used
* **Lazy Queues, Dead Letter Queues and many more….**
### Queue concept - order
Queues are FIFO manner - **in terms of producer** (messages are always held in the queue in
publication order), but **not in terms of consumer**.

Section 4.7 of the AMQP 0-9-1 core specification explains the conditions under which consuming
order is guaranteed (received in the same order that they were sent):
* messages published in one channel,
* passing through one exchange,
* stored in one queue
* consumed by exactly one outgoing channel

> For example, prioritized queue or queue where consumer rejects with requeue=true.
### Binding
* Connects Exchange with Queue
  * Using binding key (aka. Routing Key)
* Decide about message processing
  * Defines whether message posted to an exchange should be send to the queue or not
* Routing key behaviour depends from Exchange type
  * fanout - just ignores routing key,
  * topic - routing key has to be valid topic separated by dots,
  * direct - exact string match
```bash
> rabbitmqctl list_bindings
```  
## First Queue and First Consumption Hands On.
### First Queue Hands-On
* Create first queue from the web management UI.
  * Name:  
Unique queue name (Naming convention! ex: my_first_queue) in entire RabbitMQ cluster 
  * Type:  
Classic - not replicated queue
  * Durability:  
Durable - queue will survive RabbitMQ restart
  * Auto-Delete:  
    Yes - queue will be deleted after at least one consumer has connected, and then all consumers have disconnected.
* Create first message and send it into queue from the web management UI.
### First Message
  * Delivery mode  
Persistent - message will survive RabbitMQ restart
  * Headers  
AMQP level settings
  * Properties  
RabbitMQ level settings
### First consumption
  * Ack mode  
Reject, Nack(negative ack - reject extension), Ack; requeue
refers to Dead Letter Exchange (DLX)
  * Encoding   
How to handle binary payload
  * Messages   
How many messages to be read

> Requeued message is placed to its original
position **if possible**. If not message will be
requeued closer to the queue head (concurrent
deliveries and acknowledgements from
consumers).