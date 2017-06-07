varDomain = "scigility.demo"

# INSTALLATION PACKAGES LOCATION
varRepo   = "resources"
varKeytabs= "keytabs"

KAFKA_NODE_COUNT = 4
BOX="boxcutter/centos73"
ANSIBLE_RAW_SSH_ARGS = []
VAGRANT_VM_PROVIDER = "virtualbox"

## ################################################################
## start creating the machines
## ################################################################

Vagrant.configure("2") do | config |

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true  # Allow to change your local machine, for ex hosts file update
  config.hostmanager.manage_guest = true # Allow to change your guests VMs
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  ########
  ## define edge node
  ########
  config.vm.define "edge" do |subconfig|
    subconfig.vm.box = BOX
    curHostname = "edge"
    curFQDN = "#{curHostname}.#{varDomain}"
    subconfig.vm.hostname = "#{curFQDN}"
    subconfig.vm.network "private_network", ip: "10.0.0.10", :netmask => "255.255.255.0"
    subconfig.hostmanager.aliases = "edge"
    subconfig.vm.synced_folder varRepo, "/resources", id: "#{varRepo}", owner: "vagrant", group: "vagrant"
    subconfig.vm.synced_folder varKeytabs, "/keytabs", id: "#{varKeytabs}", owner: "vagrant", group: "vagrant"
    subconfig.vm.provider VAGRANT_VM_PROVIDER do |v|
      v.name = "confluent-edge"
      v.gui = false
      v.customize ["modifyvm", :id, "--memory", "2048" ]
      v.customize ["modifyvm", :id, "--cpus", "1" ]
    end
    if Vagrant.has_plugin?("vagrant-timezone")
      subconfig.timezone.value = :host
    end
    subconfig.ssh.username = "root"
    subconfig.ssh.password = "vagrant"
    #subconfig.vm.provision :shell, inline: "sed -i'' '/^127.0.0.1\t#{curHostname}/d' /etc/hosts"

  end

  ########
  ## define kafka nodes
  ########

  (1..KAFKA_NODE_COUNT-1).each do |machine_id|
    ANSIBLE_RAW_SSH_ARGS << "-o IdentityFile=.vagrant/machines/confluent-#{machine_id}/#{VAGRANT_VM_PROVIDER}/private_key"
  end

  (1..KAFKA_NODE_COUNT).each do |i|
    config.vm.define "confluent-#{i}" do |subconfig|
      subconfig.vm.box = BOX
      curHostname = "confluent-#{i}"
      curFQDN = "#{curHostname}.#{varDomain}"
      subconfig.vm.hostname = "#{curFQDN}"
      subconfig.vm.network "private_network", ip: "10.0.0.#{i + 10}", :netmask => "255.255.255.0"
      subconfig.hostmanager.aliases = "confluent-#{i}"
      subconfig.vm.synced_folder varRepo, "/resources", id: "#{varRepo}", owner: "vagrant", group: "vagrant"
      subconfig.vm.synced_folder varKeytabs, "/keytabs", id: "#{varKeytabs}", owner: "vagrant", group: "vagrant"
      subconfig.vm.provider VAGRANT_VM_PROVIDER do |v|
        v.name = "confluent-#{i}"
        v.gui = false
        v.customize ["modifyvm", :id, "--memory", "2048" ]
        v.customize ["modifyvm", :id, "--cpus", "1" ]
      end
      if Vagrant.has_plugin?("vagrant-timezone")
        subconfig.timezone.value = :host
      end
      subconfig.ssh.username = "root"
      subconfig.ssh.password = "vagrant"
      #subconfig.vm.provision :shell, inline: "sed -i'' '/^127.0.0.1\t#{curHostname}/d' /etc/hosts"


      if i == KAFKA_NODE_COUNT
        subconfig.vm.provision :ansible do |ansible|
          ansible.raw_ssh_args = ANSIBLE_RAW_SSH_ARGS
          ansible.limit = "all"
          ansible.playbook = "ansible/playbook.yml"
          #ansible.verbose = "vvv"
          ansible.groups = {
            "confluentmasters" => ["confluent-1","confluent-2"],
            "workers" => ["confluent-1","confluent-2","confluent-3","confluent-4"],
            "workers:vars" => {"kdc_master_kdc" => "edge",
                               "kdc_realm" => "scigility.demo",
                               "domain" => ".#{varDomain}",
                               "zkConnect" => "confluent-1.#{varDomain}:2181,confluent-2.#{varDomain}:2181,confluent-3.#{varDomain}:2181",
                               "schemaregistryUrl" => "http://confluent-1.#{varDomain}:8081,http://confluent-2.#{varDomain}:8081",
                               "restBootstrapServers" => "PLAINTEXT://confluent-1.#{varDomain}:9092"
                              },
            "zookeepers" => ["confluent-1","confluent-2","confluent-3"],
            "edge-nodes" => ["edge"],
            "edge-nodes:vars" => {"kdc_master_kdc" => "edge",
                                  "kdc_realm" => "scigility.demo",
                                  "kdc_database_master_key" => "admin",
                                  "kdc_admin_password" => "admin"
                                }
          }
          ansible.host_vars = {
            "confluent-1" => {"brokerid" => 1},
            "confluent-2" => {"brokerid" => 2},
            "confluent-3" => {"brokerid" => 3},
            "confluent-4" => {"brokerid" => 4}
          }

        end
      end
    end
  end

end
