# 5. RabbitMQ Common Patterns
See https://www.rabbitmq.com/tutorials
  1. Patterns Simple Queue Hands On
  2. Patterns Work Queues  Task Queues Hands On
  3. Patterns Publish  Subscribe (fanout) Hands On
  4. Patterns Publish  Subscribe based on Routing Hands On
  5. Patterns Publish  Subscribe based on Topics Hands On
  6. Patterns Publish  Subscribe based on Headers Hands On
  7. Patterns RPC - Remote Procedure Call Hands On
## Patterns Simple Queue Hands On
```mermaid
flowchart LR
    P((Producer))
    Q[[Queue]]
    C((Consumer))

    P --> Q --> C

    class P mermaid-producer
    class Q mermaid-queue
    class C mermaid-consumer
```
* https://github.com/rabbitmq/rabbitmq-website/blob/main/src/components/Tutorials/T1DiagramToC.md
* https://github.com/rabbitmq/rabbitmq-website/blob/main/tutorials/tutorial-one-java.md
* https://www.rabbitmq.com/tutorials/tutorial-one-java
## Patterns Work Queues  Task Queues Hands On
```mermaid
flowchart LR
    P((Producer))
    Q[[Queue]]
    C1((Consumer-1))
    C2((Consumer-2))

    P --> Q ;
    Q --> C1 ;
    Q --> C2 ;

    class P mermaid-producer
    class Q mermaid-queue
    class C1 mermaid-consumer
    class C2 mermaid-consumer
```
* https://github.com/rabbitmq/rabbitmq-website/blob/main/src/components/Tutorials/T2DiagramToC.md
* https://github.com/rabbitmq/rabbitmq-website/blob/main/tutorials/tutorial-two-java.md
* https://www.rabbitmq.com/tutorials/tutorial-two-java

## Patterns Publish  Subscribe (fanout) Hands On
There are a few exchange types available: ``direct``, ``topic``, ``headers`` and ``fanout``. We'll focus on the last one:
```mermaid
flowchart LR
    P((Producer))
    X{{Exchanges}}
    Q1[[Queue-1]]
    Q2[[Queue-2]]

    P --> X -- binding --> Q1 & Q2

    class P mermaid-producer
    class X mermaid-exchange
    class Q1 mermaid-queue
    class Q2 mermaid-queue
```
* https://github.com/rabbitmq/rabbitmq-website/blob/main/src/components/Tutorials/T3DiagramBinding.md
* https://github.com/rabbitmq/rabbitmq-website/blob/main/tutorials/tutorial-three-java.md
* https://www.rabbitmq.com/tutorials/tutorial-three-java
## Patterns Publish  Subscribe based on **Routing** Hands On
```mermaid
flowchart LR
    P((Producer))
    X{{Exchanges}}
    Q1[[Queue-1]]
    Q2[[Queue-2]]
    C1((Consumer-1))
    C2((Consumer-2))

    P --> X
    X -- a --> Q1 & Q2
    X -- b --> Q2
    X -- c --> Q2
    Q1 --> C1
    Q2 --> C2

    class P mermaid-producer
    class X mermaid-exchange
    class Q1 mermaid-queue
    class Q2 mermaid-queue
    class C1 mermaid-consumer
    class C2 mermaid-consumer
```
* https://github.com/rabbitmq/rabbitmq-website/blob/main/src/components/Tutorials/T4DiagramToC.md?plain=1
* https://github.com/rabbitmq/rabbitmq-website/blob/main/tutorials/tutorial-four-java.md
* https://www.rabbitmq.com/tutorials/tutorial-four-java
## Patterns Publish  Subscribe based on **Topics** Hands On
 
```mermaid
flowchart LR
    P((Producer))
    X{{Exchanges}}
    Q1[[Queue-1]]
    Q2[[Queue-2]]
    C1((Consumer-1))
    C2((Consumer-2))

    P --> X
    X -- *.orange.* --> Q1
    X -- *.*.rabbit --> Q2
    X -- lazy.# --> Q2
    Q1 --> C1
    Q2 --> C2

    class P mermaid-producer
    class X mermaid-exchange
    class Q1 mermaid-queue
    class Q2 mermaid-queue
    class C1 mermaid-consumer
    class C2 mermaid-consumer
```
* https://github.com/rabbitmq/rabbitmq-website/blob/main/src/components/Tutorials/T5DiagramTopicX.md?plain=1
* https://github.com/rabbitmq/rabbitmq-website/blob/main/tutorials/tutorial-five-java.md
* https://www.rabbitmq.com/tutorials/tutorial-five-java
## Patterns Publish  Subscribe based on **Headers** Hands On

## Patterns RPC - Remote Procedure Call Hands On
```mermaid
flowchart LR
    C((Client))
    S((Server))
    Q1[[rpc_queue]]
    Q2[[amq.gen-Xa2…]]
    Request["`Request
    reply_to=amq.gen-Xa2…
    correlation_id=abc`"]
    Reply["`Reply
    correlation_id=abc`"]

    C --- Request --> Q1 --> S --> Q2 --- Reply --> C

    class C mermaid-producer
    class Q1 mermaid-queue
    class Q2 mermaid-queue
    class S mermaid-consumer
    class Request mermaid-msg
    class Reply mermaid-msg
```
* https://github.com/rabbitmq/rabbitmq-website/blob/main/src/components/Tutorials/T6DiagramFull.md
* https://github.com/rabbitmq/rabbitmq-website/blob/main/tutorials/tutorial-six-java.md
* https://www.rabbitmq.com/tutorials/tutorial-six-java