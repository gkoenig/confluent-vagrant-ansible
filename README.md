# setting up Confluent stack via Vagrant and ansible
This repository is using Vagrant to ramp up nodes with CentOS, followed by Ansible roles to provision Zookeeper, Kafka, Kafka-Rest and Schema-Registry.  

## Prerequisites
* you have vagrant and virtualbox installed
* you have ansible installed on your host
* free resources of 8 GB RAM and 4 CPUs on your host
* reliable Internet connection, since the Confluent stack will be installed from public repository

## What will be done  

* installation of 4 nodes, based on CentOS 7 , 2 GB RAM, 1 CPU each
  * confluent-1
  * confluent-2
  * confluent-3
  * confluent-4  
* definition of the following host groups for ansible, done by [Vagrantfile](./Vagrantfile)  
  * **confluentmasters**  
  are nodes confluent-1/-2 and will get services Kafka-Rest and Schema-Registry  
  * **zookeepers**  
  include nodes confluent-1/-2/-3 and will get......guess....service Zookeeper...whooo
  * **workers**  
  includes all 4 nodes and will get Kafka Brokers installed  
* the services will be registered as _service_ on linux to be able to easily (re-)start them

## Usage
```  
git clone https://github.com/gkoenig/confluent-vagrant-ansible.git  
cd confluent-vagrant-ansible  
vagrant up
```

If you need to modify the default settings (no. of nodes, size of node, host to group mapping, ...) just open [Vagrantfile](./Vagrantfile) and adjust them accordingly.

## Hickups
the whole setup does not run absolutely flawless but installs and configures the tools in a usable manner, check [Issues](https://github.com/gkoenig/confluent-vagrant-ansible/issues) also for queued enhancements.    
Sometimes the Kafkabroker does not startup automatically, hence check it after ```vagrant up``` finished via e.g.
```
for h in confluent-1 confluent-2 confluent-3 confluent-4; do vagrant ssh $h -c "echo "" && echo $h && echo "" && service confluent-kafka status | grep -E3 'Active:'"; done
```
