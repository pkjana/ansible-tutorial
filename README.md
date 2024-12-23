# Ansible Tutorial
Ansible installation, configuration and execution on VirtualBox


# Configure the Ansible Hosts 

 1. Create a three nodes cluster in VirtualBox one of the nodes would be controller node where Ansible will be install.
 
# Configure network between hosts

 1. Open VirtualBox
 2. Select "Tools" in left pane
 3. Select "NAT Networks" in right pane
 4. Click on "Create" in right pane
 5. Select Newly Created NatNetwork
 6. Click on "Propertics" in right pane
 7. Select "General options" in the bottom of the write pane.
 8. Rename your Desire Network Name
 9. Put IP with CIDR (i.e. 192.168.100.0/24)
 10. Check DHCP
 11. Select Port Forwarding 
 12. Forward SSH port for each VM
 

# How to install ansible?

Ansible should be install in control node
$ sudo apt install ansible 


Generate ssh key on control node

for better understading generate ssh key in ~/.ssh/ansible folder instead of defauld folder.

$ ssh-keygen -t ed25519 -C "ansible"
save the key in ~/.ssh/ansible location
for better security you may use the passphrase

Now copy the public key to each of the server from control node
$ ssh-copy-id -i ~/.ssh/ansible.pub <server/node-ip>

Now try to connect the node from control node by using public key
$ ssh -i ~/.ssh/ansible <server/node-ip>

If you use ssh command without key the it will take default ssh public key if key present otherwise ask for node password

$ ssh <server/node-ip>

If you use passphrase in ssh key generation then you can avoid password typing for each time by using ssh-agent 
$ eval $(ssh-agent)   //to know pid
$ ps aux | grep <pid>
$ ssh-add
when terminal window close ssh agent forgot the key or password
$ alias ssha= 'eval $(ssh-agent) && ssh-add'
$ ssha

You can add the alias command to .bashrc


### Running ad-hoc commands ###

$ ansible all --key-file ~/.ssh/ansible -i inventory -m ping

If you put inventory file and private key file to ansible.cfg config file then no need to pass the file tin commandline
$ ansible all -m ping

List all host from inventory 
$ ansible all --list-hosts

All information about servers
$ ansible all -m gathers_facts

$ ansible all -m gathers_facts --limit <host ip>  // for a specific host

To update the all servers
$ ansible all -m apt -a update_cache=true   // it will fail because sudo is require
$ ansible all -m apt -a update_cache=true --become --ask-become-pass  // it will ask sudo password. each of the sudo password should be same

To install package on all server

$ ansible all -m apt -a name=vim-nox --become --ask-become-pass    // install vim-nox package on all server. $ apt search vim-nox

$ ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass   // $sudo apt dist-upgrade  for chrcking package is present or not.
$ ansible all -m apt -a "upgrade=dist" --become --ask-become-pass

## Playbook ##

$ nano install_apache.yaml    // this playbook helps to install apache2 package on all server
$ ansible-playbook --ask-become-pass install_apache.yaml

$ cp install_apache.yaml remove_apache.yaml
$ nano remove_apache.yaml     // To remove the apache2 package from all host
$ ansible-playbook --ask-become-pass remove_apache.yaml


