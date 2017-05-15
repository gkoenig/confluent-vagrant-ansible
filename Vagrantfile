
# Toogle start/stop agent when pleased
varDomain = "scigility.demo"

# INSTALLATION PACKAGES LOCATION
varRepo   = "~/resources"

cluster_nodes = [
  { :id => 1, :host => "confluent-1", :ram => 2048, :cpu => 1,:ip => "10.10.10.11", :box => "boxcutter/centos73", :gui => false},
  { :id => 2, :host => "confluent-2", :ram => 2048, :cpu => 1,:ip => "10.10.10.12", :box => "boxcutter/centos73", :gui => false},
  { :id => 3, :host => "confluent-3", :ram => 2048, :cpu => 1,:ip => "10.10.10.13", :box => "boxcutter/centos73", :gui => false},
  { :id => 4, :host => "confluent-4", :ram => 2048, :cpu => 1,:ip => "10.10.10.14", :box => "boxcutter/centos73", :gui => false}
]

## ################################################################
## populate /etc/hosts
## ################################################################
varHostEntries = ""
cluster_nodes.each do |cluster_node|
  varHostEntries << "#{cluster_node[:ip]} #{cluster_node[:host]}.#{varDomain} #{cluster_node[:host]}\n"
end

$etchosts = <<SCRIPT
#!/bin/bash
HOSTMANAGER=$(netstat -rn | grep "^0.0.0.0 " | cut -d " " -f10)
cat > /etc/hosts <<EOF
127.0.0.1       localhost
${HOSTMANAGER}  host.#{varDomain} host
#{varHostEntries}
EOF
SCRIPT

## ################################################################
## start creating the machines
## ################################################################

Vagrant.configure("2") do | config |

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true  # Allow to change your local machine, for ex hosts file update
  config.hostmanager.manage_guest = true # Allow to change your guests VMs
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  cluster_nodes.each do |cluster_node|
    config.vm.define cluster_node[:host] do | config |

      config.vm.provider :virtualbox do |v|
        v.name = cluster_node[:host].to_s
        v.gui = cluster_node[:gui]
        v.customize ["modifyvm", :id, "--memory", cluster_node[:ram].to_s ]
        v.customize ["modifyvm", :id, "--cpus", cluster_node[:cpu].to_s ]
      end

      config.ssh.username = "root"
      config.ssh.password = "vagrant"

      config.vm.box = cluster_node[:box]
      config.vm.hostname = "#{cluster_node[:host]}.#{varDomain}"

      config.vm.network "private_network", ip: cluster_node[:ip], :netmask => "255.255.255.0"

      # Create the /etc/hosts files for all nodes
      config.vm.provision :shell, :inline => $etchosts

      # This will added into your local machine, putting an entry on hosts file for each node, so that you address the machine via name
      # On Mac/Linux, after 'vagrant up' check your /etc/host after running. On Windowz, check %SystemRoot%\System32\drivers\etc\hosts
      config.hostmanager.aliases = "#{cluster_node[:host]}"

      config.vm.synced_folder varRepo, "/resources", id: "resources", owner: "vagrant", group: "vagrant"
    end

    if cluster_node[:id] == 4
      config.vm.provision :ansible do |ansible|
        # Disable default limit to connect to all the machines
        ansible.limit = "all"
        ansible.playbook = "ansible/playbook.yml"
        ansible.verbose = "vv"
        ansible.groups = {
          "confluentmasters" => ["confluent-1","confluent-2"],
          "workers" => ["confluent-1","confluent-2","confluent-3","confluent-4"],
          "zookeepers" => ["confluent-1","confluent-2","confluent-3"]
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
