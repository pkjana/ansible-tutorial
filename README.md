# Ansible Tutorial
This tutorial regarding Ansible installation, configuration and execution on VirtualBox. It is created during practice session of [Ansible course](https://www.youtube.com/playlist?list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70) by Learn Linux TV


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
Save the key in ~/.ssh/ansible location for better security you may use the passphrase

Now copy the public key to each of the server from control node.\
$ ssh-copy-id -i ~/.ssh/ansible.pub <server/node-ip>

Now try to connect the node from control node by using public key.\
$ ssh -i ~/.ssh/ansible <server/node-ip>

If you use ssh command without key the it will take default ssh public key if key present otherwise ask for node password

$ ssh <server/node-ip>

If you use passphrase in ssh key generation then you can avoid password typing for each time by using ssh-agent 

$ eval $(ssh-agent)   //to know pid\
$ ps aux | grep <pid>\
$ ssh-add

when terminal window close ssh agent forgot the key or password

$ alias ssha= 'eval $(ssh-agent) && ssh-add'\
$ ssha

You can add the alias command to .bashrc


# Running ad-hoc commands

$ ansible all --key-file ~/.ssh/ansible -i inventory -m ping

If you put inventory file and private key file to ansible.cfg config file then no need to pass the file tin commandline.\
$ ansible all -m ping

List all host from inventory \
$ ansible all --list-hosts

All information about servers\
$ ansible all -m gather_facts

$ ansible all -m gather_facts --limit <host ip>  // for a specific host

To update the all servers\
$ ansible all -m apt -a update_cache=true      // it will fail because sudo is require\
$ ansible all -m apt -a update_cache=true --become --ask-become-pass     // it will ask sudo password. each of the sudo password should be same

To install package on all server

$ ansible all -m apt -a name=vim-nox --become --ask-become-pass    // install vim-nox package on all server. $ apt search vim-nox

$ ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass   // $sudo apt dist-upgrade  for chrcking package is present or not.\
$ ansible all -m apt -a "upgrade=dist" --become --ask-become-pass

# Playbook 

$ nano install_apache.yaml    // this playbook helps to install apache2 package on all server\
$ ansible-playbook --ask-become-pass install_apache.yaml

$ cp install_apache.yaml remove_apache.yaml\
$ nano remove_apache.yaml     // To remove the apache2 package from all host\
$ ansible-playbook --ask-become-pass remove_apache.yaml

For CentOS server httpd server ststus inactive by default. So for the time being we need to start the httpd server manually by following commands then it will show default Http Server Page in web browser.\
$ sudo systemctl status httpd\
$ sudo systemctl start httpd\
$ sudo firewall-cmd --add-port=80/tcp

# How to individulalized our host and install different packages in different categories of servers. 

Modify ansible config file by using this **inventory_specific_nodes** inventory file while executing the **install_pkg_on_specific_nodes.yaml** playbook.

Check the DB server status using following command.\
$ systemctl status mariadb

Check the file server status using following command.\
$ systemctl status smbd

# Tags

Ansible tags serve as markers that can be added to individual tasks within an Ansible playbook, enabling the execution of specific or designated tasks based on these tags during playbook runs. You'll occasionally need to execute only a portion of a playbook rather than the whole thing.

Show the available tags in the playbook.\
$ ansible-playbook --list-tags install_pkg_by_tags.yaml   // to execute the playbook use the inventory **inventory_specific_nodes**

$ ansible-playbook --tags centos --ask-become-pass install_pkg_by_tags.yaml

$ ansible-playbook --tags db --ask-become-pass install_pkg_by_tags.yaml

$ ansible-playbook --tags apache --ask-become-pass install_pkg_by_tags.yaml

$ ansible-playbook --tags "apache,db" --ask-become-pass install_pkg_by_tags.yaml   // execute multiple tags

# Managing Files

$ ansible-playbook  --ask-become-pass managing_files.yaml   // to execute the playbook use the inventory inventory_specific_nodes

To testing the web page call the web server ip address from browser or use curl command in CLI.\
$ curl 192.168.100.9

# Managing Services

There are lots of services manage by Ansible. Here three services are covered in the managing_services.yaml playbook regarding webserver start, restart, config change.

Check the status of the httpd server\
$ sudo systemctl status httpd

If it is in running state then stop it and verify it in web browser.\
$ sudo systemctl stop httpd

Check the status again of the httpd server. It should be in inactive state and service may be desabled. If the service is desabled while server rebootted then server will not be start automatically. \
$ sudo systemctl status httpd
