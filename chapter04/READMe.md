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
## Configuration
## Configuration file Hands On
* https://github.com/rabbitmq/rabbitmq-server/blob/master/deps/rabbit/docs/rabbitmq.conf.example

## Plugins