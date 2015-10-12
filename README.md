# vagrant-chef-full
contains a chefserver, a chefworkstation, and 3 nodes

# what to do
`Vagrant up` will create  all the machines.  
Once this is done you need to:
 - Download the Starter Kit
    - From a web browser, navigate to your Chef server's hostname over HTTPS, https://127.0.0.1:4433.
    - Sign in using admin admin.
    - When prompted, enter the full name (for example, Web development team) and short name (for example, webdev) for your organization.
    - From the Administration tab, select your organization.
    - Select Starter Kit from the menu on the left.
    - Click the Download Starter Kit button.
    - Click Proceed. Save the file chef-starter.zip to your computer.
    - Extract chef-starter.zip to the chef-repo directory.
 - Now verify that the chef-repo/.chef directory on your workstation contains knife configuration file and your RSA key.


```
# Connect on your workstation
vagrant ssh chefworkstation

# Enter the chef-repo folder
cd chef-repo

# Download the Chef server's SSL certificate
knife ssl fetch

# Verify that the certificate was properly retrieved and can be used to authenticate calls to the Chef server.
knife ssl check

# Test the connection to Chef server
knife client list

```
