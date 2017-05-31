# setting up Confluent stack via Vagrant and ansible
This repository is using Vagrant to ramp up nodes with CentOS, followed by Ansible roles to provision MIT-Kerberos, Zookeeper, Kafka, Kafka-Rest and Schema-Registry.  

## Prerequisites
* you have vagrant and virtualbox installed
* you have ansible installed on your host
* free resources of 8 GB RAM and 4 CPUs on your host

## What will be done  
by using this repo without any modification, the following activities will be performed:  

* installation of nodes, based on CentOS 7 , 2 GB RAM, 1 CPU each
  * confluent-1
  * confluent-2
  * confluent-3
  * confluent-4
  * edge
* definition of the following host groups for ansible, done by [Vagrantfile](./Vagrantfile)  
  * **confluentmasters**  
  are nodes confluent-1/-2 and will get services Kafka-Rest and Schema-Registry  
  * **zookeepers**  
  include nodes confluent-1/-2/-3 and will get......guess....service Zookeeper...whooo
  * **workers**  
  includes all 4 non-edge-nodes and will get Kafka Brokers installed  
* the Confluent stack will be installed into target dir ```/opt/confluent``` by using the tar.gz from within [this repo](./resources)

## Getting started
```  
git clone https://github.com/gkoenig/confluent-vagrant-ansible.git  
cd confluent-vagrant-ansible  
vagrant up
```
- It will take some minutes to startup, especially the creation of the Kerberos database.  
- If you are facing issues while downloading the confluent tar.gz from GitHub LFS , try [this public URL](http://packages.confluent.io/archive/3.2/confluent-oss-3.2.1-2.11.tar.gz) to fetch Confluent stack v3.2.1  
- If you need to modify the default settings (no. of nodes, size of node, host to group mapping, ...) just open [Vagrantfile](./Vagrantfile) and adjust them accordingly.


## Services startup
after successfull creation of the nodes, the services are **_not_ started automatically**. Hence you have to start it in order by using the scripts in ```/opt/confluent/bin```. Order:
1. zookeeper-server-start
2. kafka-server-start
3. Schema-Registry and REST-Proxy, if required    

### Zookeeper
Before starting Zookeeper, the corresponding _jaas.conf file needs to be exported as appropriate env. variable on each of the Zookeeper servers, followed by the actual start script:
```
source /etc/kafka/zookeeper-env.sh
/opt/confluent/bin/zookeeper-server-start /etc/kafka/zookeeper.properties
```
To start Zookeeper in **daemon mode**, just add the ```-daemon``` parameter as shown below:  
```
source /etc/kafka/zookeeper-env.sh
/opt/confluent/bin/zookeeper-server-start -daemon /etc/kafka/zookeeper.properties
```

### Kafka Brokers
Similar to Zookeeper, also the Kafka Brokers needs to know the location of the _jaas.conf file, where authentication details are specified. Since this environment offers Kerberos auth. to Kafka, this file needs to be known. Startup procedure (on each of the Kafka nodes):  
```
source /etc/kafka/kafka-env.sh
/opt/confluent/bin/kafka-server-start /etc/kafka/server.properties
```

## Client interaction examples
...tbd...
