# check_lenovo_xclarity
nagios check for Lenovo xClarity service processors

Nagios plugin for checking hardware health of Lenovo servers via xClarity Controller (XCC) out of band management using SSH

# Preparation
Connect the xClarity ethernet interface to the network and give it an IP address
<br><img src=images/xclarity_interface.png>

# Create userid on xClarity 
Login to the xClarity web interface
Click BMC Configuration, User/LDAP
<br><img src=images/xclarity_bmc.png>

Create a userid called nagios with the Read-only authority level.
Paste the contents of the $HOME/.ssh/id_rsa.pub file from the nagios server into the text field.  This will allow the nagios userid on the nagios server to SSH into the xClarity port to retrieve health metrics.
<br><img src=images/xclarity_sshkey.png>


This script assumes that SSH key pairs are in place to allow the nagios server to connect to the xClarity Controller via SSH

You will need to add the following section to commands.cfg on the nagios server:
```
    # 'check_lenovo_xclarity' command definition
    define command{
            command_name    check_lenovo_xclarity
            command_line    $USER1$/check_lenovo_xclarity -H $HOSTADDRESS$
            }
 ```
 
 
You will need to add the following section to services.cfg on the nagios server:
```
    # Define a service to check the health of Lenovo xClarity controllers
    define service{
           use                             generic-24x7-service
           hostgroup_name                  all_lenovo_xclarity
           service_description             xClarity health
           check_command                   check_lenovo_xclarity
           }
```

TROUBLESHOOTING
---------------
1) Ensure that SSH key pairs are configured correctly by attempting to SSH into the remote host as the nagios user.
   If you get prompted for a password, SSH key pair authentication is not working.
   Login to the xClarity controller, click BMC Configuration, User/LDAP, create a user called nagios, paste in contents of $HOME/.ssh/id_rsa.pub 


You should see results similar to the following:
<img src=images/check_lenovo_xclarity.png>
