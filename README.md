# Creates a slurm cluster in pouta

Tested with slurm versions:
 - 14.11.0
 - 14.11.3
 - 15.08.0

## How-To

### Launch the Openstack instances:

 - ansible-playbook launcopenstackinstance.yml # launches the VMs. source the openstack-rc script before running this playbook. Also update the playbook to include the names of your key and tenant.
  - see the group_vars/all/all file for default variables used for launching an OS instance

### Initial configuration of the instances and your workstation:

 - First yum -y install nc on the bastion host.
 - Then setup SSH config so you don't have to have a public IP on each instance. Change the Hostname in "Host bastion" to the service node.

Put this in ~/.ssh/config :

<pre>
# http://edgeofsanity.net/article/2012/10/15/ssh-leap-frog.html
# This applies to all hosts in your ssh config.
ControlMaster auto
ControlPath ~/.ssh/ssh_control_%h_%p_%r

# Always ssh as cloud-user and don't save hostkeys
Host bastion
  User cloud-user
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  Hostname 86.50.168.39
     
Host slurm* 
   User cloud-user
   StrictHostKeyChecking no
   UserKnownHostsFile /dev/null
   ForwardAgent yes
   ProxyCommand ssh bastion nc %h %p
</pre>

 - Second update the /etc/hosts on the service node (playbook should update the others):

</pre>
# For second cluster
192.168.36.126 slurm2-compute1
192.168.36.125 slurm2-service
192.168.36.124 slurm2-login

# For first cluster
192.168.36.129 slurm-compute1
192.168.36.128 slurm-service
192.168.36.127 slurm-login
</pre>

### Then we can finally run the slurm configuration playbooks:

\_ Update the files in group_vars/ to your settings
 
You also need to add a mysql_slurm_password: "PASSWORD" string somewhere. This will be used to set a password for the slurm mysql user.


#### Description of the playbooks:

 - site.yml - calls the slurm*.yml playbooks
 - slurm_*.yml # The playbooks that configure the servers
  - set slurm_version - this is used to determine which version to download from schedmd.com

#### Run them in this order:

 - Update stage to have the right IP addresses and hostnames.
 - configuring 1st slurm: ansible-playbook site.yml

 - Update stage to have the right IP addresses and hostnames.
 - configuring 2st slurm: ansible-playbook site.yml

#### Add cloud-user to slurm

<pre>
sacctmgr create account name=csc
sacctmgr create user name=cloud-user account=csc
</pre>

### Make changes to slurm.conf and distribute it to nodes and restart/reconfigure:

 - Would be nice with a role /tag where one could just run ansible-playbook site.yml --tag new-slur-config and it pushes new config and restarts/reconfigs as necessary.

# Authors:

 - Marco Passerini (original author)
 - Johan Guldmyr (updates done as part of FGCI work)
