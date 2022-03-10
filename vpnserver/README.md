# OpenVPN deployment and management with Ansible
These playbooks can be used to install and configure OpenVPN as well as create and revoke vpn client configs. It has been tested on CentOS 7 and Debian 11.  

As a note of caution, this will produce self-signed certificates and should only be used for internal testing. 
A more secure system will have certificates signed by a seperate Certificate Authority.

The target host requires SSH keys copied to both root and a non-root admin account with sudo privilages ("vpnadmin" by default). 
The vpnadmin account can optionally be created using the playbook "admin_create.yml".
Both public and private keys should be placed in the local './keys' directory of the control node and the path to the private key set in 'ansible_cfg':


``` 
[defaults]
private_key_file = ./keys/<id_rsa>
```

# Using admin_create.yml to create vpnadmin account (optional):
If desired, the admin account 'vpnadmin' can be created using the 'admin_create.yml' playbook. This still requires root ssh acces.
A password hash must be passed to the playbook as an extra variable:

```
hashout=$(mkpasswd --method=sha-512)
... <enter password>
ansible-playbook -i hosts playbooks/admin_create.yml --extra-vars "hash=$hashout"
```


# Main OpenVPN install and config
```ansible-playbook -K -i hosts playbooks/main.yml```

By default, this will not configure the vpn to have clients push all traffic through the vpn. 
To enable this, uncomment line 181: 

```
#- { FROM: ';push "redirect-gateway def1 bypass-dhcp"', TO: 'push "redirect-gateway def1 bypass-dhcp"' }
```

To enable the vpn server to set the client's dns, uncomment lines 183-184 and replace DNS addresses:

```
#- { FROM: '^;push "dhcp-option DNS "', TO: 'push "dhcp-option DNS 1.1.1.1."' }
#- { FROM: '^;push "dhcp-option DNS "', TO: 'push "dhcp-option DNS 1.0.0.1."' }
```
 


# Generating OpenVPN client configs
To create a client config file and store it locally in the the ./clientconfs/ directory
(Replace ```<client>``` with intended client name):

```ansible-playbook -K -i hosts playbook/client_create.yml --extra-vars "client_name=<client>"```

 
This can now be run on the client as:
 ```openvpn --config <config>```

# Revoking client certificates
```ansible-playbook -K -i hosts playbook/client_revoke.yml --extra-vars "client_name=<client>"```

Once revoked, the client will lose connection and the generated client config will no longer successfully authenticate.
