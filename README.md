# check_lenovo_xclarity
nagios check for Lenovo xClarity service processors

Nagios plugin for checking hardware health of Lenovo servers via xClarity Controller (XCC) out of band management using SSH

# Requirements
perl, low-privilege read-only user account on xClarity, SSH key pair auth between nagios server and xClarity

# Preparation
Connect the xClarity ethernet interface to the network and give it an IP address
<br><img src=images/xclarity_interface.png>

# Create userid on xClarity (CLI method)
SSH into the xClarity with the built-in USERID account.
Check to see what users already exist.  In this example, only the default USERID account exists, which means the next available numeric account id will be 2.
```
system> users
 Account      Login ID      Advanced Attribute                       Access            Password Expires
 -------      --------      ------------------                       ------            ----------------
    1           USERID                  Native                   Read/Write     Password doesn't expire
```

Create the new userid with numeric index 2 (adjust as appropriate if your xClarity already has more users)
```
system> users -2 -n nagios -p SecretPass -a ro
```

Grab the SSH public key from the nagios server.
In this example, -2 refers to the numeric index of the user account, and -1 refers to the numeric key_index for the SSH public key.
```
system> users -2 -pk -upld -1 -i nagios01.example.com -u SomeUserName -pw SomePassword -l /path/to/id_rsa.pub
ok
```

Confirm the SSH public key for the newly created nagios user on the xClarity is visible:
```
system> users -2 -pk -e -1
ssh-rsa AAAAB3NzaC1yc____key_truncated___4exHCpN nagios@nagios01.example.com
```

At this point, you should be able to SSH from the nagios server to the xClarity without a password and run commands:
```
[nagios@nagios01.example.com~]$ ssh server-xclarity.example.com
system> vpd sys
Machine Type-Model             Serial Number                  UUID
--------------                 ---------                      ----
7Z46CTO1XX                     J100VXXX                       7D7D6AFE4D9511EAB0B80A3A88XXXXXX
```

# Create userid on xClarity (GUI method)
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
