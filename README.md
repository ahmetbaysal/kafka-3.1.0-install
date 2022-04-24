# apache-kafka-installation-on-ubuntu20.04
* official-web-site : https://kafka.apache.org/
* download-latest-version : https://kafka.apache.org/downloads

## Installation

Kafka requires [Java](https://www.oracle.com/java/) to run.
#### 1-  Install Java
We can install Java via apt-get install.
```sh
sudo apt-get update
sudo apt-get install openjdk-11-jdk
```
Find installed java directory.
```sh
update-alternatives --config java
```
*  /usr/lib/jvm/java-11-openjdk-amd64/bin/java

We should add JAVA_HOME and Path variables to java.sh file
```sh
sudo vi /etc/profile.d/java.sh
```
Add these lines into java.sh
```sh
#/bin/bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
```
Apply changes
```sh
source /etc/profile.d/java.sh
```
Check Java version
```sh
java --version 
```
#### 2-  Install Apache Kafka
We have 2 opitons;
1- Download via wget : 
```sh 
sudo wget https://dlcdn.apache.org/kafka/3.1.0/kafka_2.13-3.1.0.tgz
```
2- Download manuel tgz file from web site and then move file to where you want to install directory: 
```sh 
https://dlcdn.apache.org/kafka/3.1.0/kafka_2.13-3.1.0.tgz
```
Extract tgz file 
```sh 
sudo tar xzf kafka_2.13-3.1.0.tgz 
```
Move file where you want.
```sh 
sudo mv kafka_2.13-3.1.0 /data/
```
Create 2 folder for logs.
```sh 
cd /data/kafka_2.13-3.1.0/
mkdir zookeeper
mkdir kafka-logs
```
Go inside kafka config directory. We should change a few properties.
```sh
cd kafka_2.13-3.1.0/config/
```
```sh
sudo nano zookeeper.properties
```
* change this path like 
* * dataDir=/data/kafka_2.13-3.1.0/zookeeper
* Hint : zookeeper default port is 2181 but i have used it, therefore i should change
* * clientPort=5181
```sh
sudo nano server.properties
```
* change this path like 
* * log.dirs=/data/kafka_2.13-3.1.0/kafka-logs
* Hint : I should change zookeeper connection 
* * zookeeper.connect=localhost:5181

We completed properties. Now we can start zookeeper and kafka. Lets start with zookeeper.

Go inside kafka bin directory
```
cd ..
cd bin
```
Run Zookeeper
```
nohup sudo ./zookeeper-server-start.sh /data/kafka_2.13-3.1.0/config/zookeeper.properties &
```
Open new terminal then run Kafka Broker
```
nohup sudo ./kafka-server-start.sh /data/kafka_2.13-3.1.0/config/server.properties &
```
Zookeeper and Kafka should be running mode. If you have a problem you should restart everyone and start new setup.


Create Topic 
```
./kafka-topics.sh --create --topic myTopic --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1
```
Publish first message 
```
echo "Hello topic!!" | ./kafka-console-producer.sh --topic myTopic --bootstrap-server localhost:9092
```
Read message with consumer
```
./kafka-console-consumer.sh --topic myTopic --from-beginning --bootstrap-server localhost:9092
```
OPTIONAL - Kafka run as a system service
```
cd /etc/systemd/system
sudo vi zookeeper.service
```
```
[Unit]
Description=Bootstrapped Zookeeper
After=syslog.target network.target
[Service]
Type=simple
User=CHANGE_USER
Group=CHANGE_USER
ExecStart=/data/kafka_2.13-3.1.0/bin/zookeeper-server-start.sh /data/kafka_2.13-3.1.0/config/zookeeper.properties
ExecStop=/data/kafka_2.13-3.1.0/bin/zookeeper-server-stop.sh
[Install]
WantedBy=multi-user.target
```
```
sudo vi kafka.service
```
```
[Unit]
Description=Apache Kafka
Requires=zookeeper.service
After=zookeeper.service
[Service]
Type=simple
User=CHANGE_USER
Group=CHANGE_USER
ExecStart=/data/kafka_2.13-3.1.0/bin/kafka-server-start.sh /data/kafka_2.13-3.1.0/config/server.properties
ExecStop=/data/kafka_2.13-3.1.0/bin/kafka-server-stop.sh
[Install]
WantedBy=multi-user.target
```


```
sudo systemctl daemon-reload  #notifies the system about the changes in the configurations

sudo systemctl enable zookeeper.service 
sudo systemctl enable kafka.service
sudo systemctl start zookeeper.service
sudo systemctl start kafka.service

sudo systemctl status zookeeper.service
sudo systemctl status kafka.service

sudo systemctl stop zookeeper.service
sudo systemctl stop kafka.service
```

