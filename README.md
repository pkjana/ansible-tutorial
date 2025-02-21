# Ansible Tutorial
This tutorial regarding Ansible installation, configuration and execution on VirtualBox. It is created during practice session of [Ansible course](https://www.youtube.com/playlist?list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70) by Learn Linux TV


# Configure the Ansible Hosts 

 1. Create a three nodes cluster in VirtualBox one of the nodes would be controller node where Ansible will be install.
 
# Configure network between hosts
 
 ## Configure network adapter
 
 1. Open VirtualBox
 2. Select "Tools" in left pane 
 3. Select Host-only Networks
 4. Click on "Create" in right pane
 5. Select Newly Created Host-only Network
 6. Click on "Properties" in right pane
 7. Select "Configure Adapter Manually" in the Adapter Tab from the bottom pane.
 8. Put IP Address and netmask manually ( i.e IP: 192.168.60.1 Netmask: 255.255.255.0)
 9. Don't enable DHCP
 
 ## Configure network in VM
 
 1. Select VM
 2 Click Settings on right pane
 3. Select Network
 4. Adapter 1 attached to NAT   // it is for internet
 5. Adapter 2 attached to Host-only Adapter and choose your pre configured Host-only adapter name
 
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


# User Management

1. Create the user 
2. Add authorize key module to the user
3. Add sudoers profile to the user

In the *user_management.yaml* above three task are covered. Use inventory_specific_nodes inventory file in ansible config file to run this playbook. Dont forget to modify your public key and you may change your desire user name in the proper task.     

Before running the playbook you can check the your desire name in the passwd file.\
$ cat /etc/passwd

If we add a users file to /etc/sudoers.d directory with the sudoer syntax we are able to control access to sudo 
$ ls -l /etc/sudoers.d/

$ ssh -i ~/.ssh/ansible simone@`<node-IP>`

As we can see I was logged in immediately to that server without no passphrase or no password.\
$ whoami  // it should show the user name simone.
$ sudo apt update  // do not get any password prompt
$ cat .ssh/authorize_keys  //it will show authorize_keys added to the user
$ sudo cat /etc/sudoers.d/simone    // to show the sudoer profile syntax of the user simone


## Password less  playbook execution

As we in times of executing playbook it is asking server password.
$ ansible-playbook --ask-become-pass <playbook file>

To over come this problem we need to add following text to ansible config file\
remote_user=`<user-name>`  // here user name is simone. 

Now we run playbook with user simone. There no need flag --ask-become-pass
$ ansible-playbook <playbook file>

## Simplify and segregate the playbook

when we are setting up a fresh server we are not going to have the simone user on that server. So what we generally like to do is create a bootstrap playbook and in that playbook i will put all the commands in there that are required to initility set up server with the users and anything that's absolutely required to get ansible itself provisioned. 

In the bootstrap playbook we install OS update, create user, add ssh public key to that user and sudoer file to the user. So there is a initial playbook called  bootstrap.yaml. Before executing the bootstrap playbook you need to comment out the remote_user line in the ansible.cfg file. Otherwise the playbook will not create the user.
$ ansible-playbook --ask-become-pass bootstrap.yaml


Now we have our desire user name (simone) with public key. So, we simplify the user_management.yaml and copied it to user_management_post_bootstrap.yaml.

$ cp user_management.yaml user_management_post_bootstrap.yaml

Before executing the post bootstrap playbooks you need to uncomment the remote_user line in the ansible.cfg file.\
$ ansible-playbook user_management_post_bootstrap.yaml  // become password is no longer needed.

We did not erase the "Add ssh key for simone" task in the user_management_post_bootstrap.yaml for future key management. 

# Roles

Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content into roles, you can easily reuse them and share them with other users. 

## Role directory structure
An Ansible role has a defined directory structure. Here we have created roles for tasks. So, we created *roles* directory then create role name dir (i.e. web_servers, etc) under it , under role name need to create *tasks* directory and under tasks need to create *main.yaml*

Use *inventory_specific_nodes* inventory file in ansible config file to run this *roles_management.yaml* playbook.

$ ansible-playbook roles_management.yaml


