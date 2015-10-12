# vagrant-chef-full
contains a chefserver (opscode-manage/opscode-reporting), a chefworkstation, and 3 nodes

# what to do
`Vagrant up` will create  all the machines.  
Once this is done you need to:
 - Download the Starter Kit
    - From a web browser, navigate to your Chef server's hostname over HTTPS
      - https://127.0.0.1:4433/login (Upon form validation, it will redirect you to a 404 not found)
      - then manually go to https://127.0.0.1:4433/
    - Sign in using admin password.
    - When prompted, enter the full name (for example, Web development team) and short name (for example, webdev) for your organization.
    - From the Administration tab, select your organization.
    - Select Starter Kit from the menu on the left.
    - Click the Download Starter Kit button.
    - Click Proceed. Save the file `chef-starter.zip` to your computer.
    - Extract `chef-starter.zip` to the `chef-repo` directory.
 - Now verify that the `chef-repo/.chef` directory on your workstation contains knife configuration file and your RSA key.


```
# Connect on your workstation
vagrant ssh chefworkstation

# Enter the chef-repo folder
cd sources/chef-repo

# Download the Chef server's SSL certificate
knife ssl fetch

# Verify that the certificate was properly retrieved and can be used to authenticate calls to the Chef server.
knife ssl check

# Test the connection to Chef server
knife client list


## Some commands
## Update a Cookbook
knife cookbook upload hello_chef_server
## Bootstrap a node
knife bootstrap 192.168.10.98 --ssh-user debian --sudo --identity-file ../vagrant_keys/id_rsa --node-name node1
# sudo user password: 78yu[]OP
## Update a node (by executing it's recipe)
knife ssh 192.168.10.98 'sudo chef-client' --manual-list --ssh-user debian --identity-file ../vagrant_keys/id_rsa
# sudo user password: 78yu[]OP
## Excuting a command on a specific node
## Since there is no DNS hostname resolution we need to use --ssh-gateway with the node ip
knife ssh "name:adwordslayer" "sudo chef-client" -x debian --identity-file ../../../vagrant_keys/id_rsa --ssh-gateway 192.168.10.98


```

# Upon success

Upon success a message like this one should appear:

```console
==> chefserver: Machine 'chefserver' has a post `vagrant up` message. This is a message
==> chefserver: from the creator of the Vagrantfile, and not from Vagrant itself:
==> chefserver:
==> chefserver: chefserver is now live...
==> chefserver:     Public IP: 192.168.10.100
==> chefserver:     Private IP: 192.168.10.254
==> chefserver:     Machine ports : 443,80
==> chefserver:     Host ports : 4433,8003
==> chefserver:         Chef server now accessible at https://127.0.0.1:4433/
==> chefserver:     Admin username => admin
==> chefserver:     Admin password => password

==> chefworkstation: Machine 'chefworkstation' has a post `vagrant up` message. This is a message
==> chefworkstation: from the creator of the Vagrantfile, and not from Vagrant itself:
==> chefworkstation:
==> chefworkstation: chefworkstation is now live...
==> chefworkstation:     Public IP: 192.168.10.99
==> chefworkstation:     Private IP: 192.168.10.253
==> chefworkstation:     Machine ports :
==> chefworkstation:     Host ports :

==> node1: Machine 'node1' has a post `vagrant up` message. This is a message
==> node1: from the creator of the Vagrantfile, and not from Vagrant itself:
==> node1:
==> node1: node1 is now live...
==> node1:     Public IP: 192.168.10.98
==> node1:     Private IP: 192.168.10.252
==> node1:     Machine ports : 80,443
==> node1:     Host ports : 8002,4432
```
