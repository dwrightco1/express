# Platform9 Autodeplopy
Autodeploy aims to automate the prerequisite tasks required to bring Openstack hypervisors and Kubernetes containervisors under management by a Platform9 control plane, including package/service prerequisites, host agent(s), and control plane authorization.

GitHub Repository : [https://github.com/platform9/autodeploy.git](https://github.com/platform9/autodeploy.git)

## Installation/Setup Instructions

**Step 1 : Run Setup**
```
$ ./INSTALL -s
Instance URL: https://acme.platform9.horse
--> accepted: https://acme.platform9.horse

Admin Username: admin@platform9.net
--> accepted: admin@platform9.net

Admin Password: ---masked---
--> accepted: ---masked---

Region: master
--> accepted: master

Tennant [service]: admin
--> accepted: admin

Manage Hostname [true false] [false]:
--> accepted: false

Manage DNS Resolver [true false] [false]:
--> accepted: false

DNS Resolver 1 [8.8.8.8]:
--> accepted: 8.8.8.8

DNS Resolver 2 [8.8.4.4]:
--> accepted: 8.8.4.4

DNS Domain for Nova Hypervisors: cs.platform9.net
--> accepted: cs.platform9.net

Proxy URL:
--> accepted: -
```

**Step 2 : Configure Your Inventory**
* vi inventory/hosts 

NOTE: The above file is a sample starting point, with a reference configuration for both Openstack and Kubernetes. You'll need change the hostnames and IP addresses to reflect your environment.

* Ansible Inventory Example
```
##
## Ansible Inventory
##
[all]
[all:vars]
ansible_ssh_pass=Pl@tform9
ansible_sudo_pass=Pl@tform9

################################################################################################
## Openstack Groups
################################################################################################
## global variables defined in group_vars/hypervisors.yml
[hypervisors]
hv10 ansible_host=172.16.7.172 ansible_user=centos ha_cluster_ip=172.16.7.172 dhcp=on snat=on glance=on
hv11 ansible_host=172.16.7.171 ansible_user=ubuntu ha_cluster_ip=172.16.7.171

## global variables defined in group_vars/glance.yml
[glance]
hv10

################################################################################################
## Kubernetes Groups
################################################################################################
## global variables defined in group_vars/containervisors.yml
[containervisors]
cv01 ansible_host=172.16.7.116 ansible_user=centos cluster_name=c1 cluster_fqdn=c1.platform9.netcv02 ansible_host=172.16.7.88 ansible_user=centos cluster_name=c1 cluster_fqdn=c1.platform9.net
```

**Step 3: Run Auto-Deploy**
* ./INSTALL [-a] \<target\>

Where '\<target\>' is a hostname or group defined in Inventory file.

NOTE: if you include the '-a' flag, Autodeploy will perform pre-authorization and role deployment for the hypervisor or containervisor.

## Usage Notes
Here's the usage statement for the Autodeploy installer:
```
$ ./INSTALL
Usage: ./INSTALL [Args] <target>

Args (Optional):

-a|--autoRegister        : auto-register host with control plane
-i|--installPrereqs      : install pre-requisites and exit
-s|--setup               : run setup and exit
-c|--config <configFile> : use custom configuration file
-e|--extra-vars <string> : ansible extra-vars <name=val,...>
-h|--help                : display this message
```

**Managing Multiple DUs**
If you have more than one Platform9 DU to manage, you can create a configuration file for each one (using pf9-autodeploy.conf as a template) and start INSTALL with the '-c' flag:
```
./INSTALL -c ~/pf9-site1.conf -a hv01
```

**Overriding Inventory Variable**
If you want to override an Ansible variable defined in Inventory or dynamically within playbooks, you can invoke INSTALL with the '-e' flag:
```
./INSTALL -c ~/pf9-autodeploy.conf -a -e "proxy_url=https://proxy1.platform9.net" hv01
```
NOTE: Variables passed as extra-vars have the highest precedence.

## License

Commerical

## To-Do's
ovs-vsctl del-port br-pf9 bond0
ovs-vsctl add-port br-pf9 bond0.207

/etc/sysconfig/network-scripts/ifcfg-bond0:
DEVICE=bond0
ONBOOT=yes
BONDING_MASTER=yes
BONDING_OPTS="mode=0"
MTU=1600

/etc/sysconfig/network-scripts/ifcfg-bond0.207:
BOOTPROTO=none
DEVICE=bond0.207
VLAN=yes
ONBOOT=yes
TYPE=vlan
MASTER=bond0
SLAVE=yes

-------------------------------------------------------
Working Config
-------------------------------------------------------
[root@hv102 network-scripts]# cat ifcfg-ens160.0
# Created by cloud-init on instance boot automatically, do not edit.
#
VLAN=yes
TYPE=Vlan
PHYSDEV=ens160
VLAN_ID=0
BOOTPROTO=none
DEFROUTE=yes
DEVICE=ens160
GATEWAY=172.16.7.1
HWADDR=fa:16:3e:c8:6f:1b
IPADDR=172.16.7.195
NETMASK=255.255.255.0
ONBOOT=yes
TYPE=Ethernet
USERCTL=no
NM_CONTROLLED=no
[root@hv102 network-scripts]# cat ifcfg-ens160.207
# Created by cloud-init on instance boot automatically, do not edit.
#
VLAN=yes
TYPE=Vlan
PHYSDEV=ens160
VLAN_ID=207
BOOTPROTO=none
DEVICE=ens160
ONBOOT=yes
TYPE=Ethernet
USERCTL=no
NM_CONTROLLED=no
