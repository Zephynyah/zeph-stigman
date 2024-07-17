# Reboot and validation

Perform the following procedure below to reboot the Host Server and validate previous configurations

## Procedures
* Perform and Host Server reboot
* Validate Keycloak services is still running 
* Validate STIG Manager services is still running 
* Validate Nginx services is still running 
* Validate Temporarily enabled port are not open
* Validate Permanently enabled Services are still open


---
## Perform and Host Server reboot
Execute the below commands to reboot the computer.
```sh
reboot
```

Wait for the server to reboot the log back into the shell.

---
## Validate Keycloak services is still running 
Execute the below commands to check the status of the service:
```sh
sudo systemctl status keycloak.service
```

---
## Validate STIG Manager services is still running 
Execute the below commands to check the status of the service:
``` bash
sudo systemctl status stigman.service
```

---
## Validate Nginx services is still running 
Execute the below commands to check the status of the service:
``` bash
sudo systemctl status nginx.service
```

---
## Validate Temporarily enabled port are not open

Execute the below commands to check  for open ports and services.
```sh
sudo firewall-cmd --zone=public --permanent --list-ports
```

!!! info
    Port `8080/tcp` and `54000/tcp` should not be present in this output. 

Or using the list-all command line option
```sh
sudo firewall-cmd --list-all
```

!!! info
    Open ports are listed on line starting with ports: Port `8080/tcp` and `54000/tcp` should not be present in this output. 

Output:
```{.console .no-copy}
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: http https ssh
  ports: 
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

---
## Validate Permanently enabled Services are still open

Execute the below commands to check  for open ports and services.
```sh
sudo firewall-cmd --zone=public --permanent --list-services
```
!!! info
    `http` and `https` service should be present in this output. 

Output:
```sh
http https ssh
```

Or using the list-all command line option
```sh
sudo firewall-cmd --list-all
```

!!! info
    Open services are listed on line starting with services: `http` and `https` Service should be present in this output. 

Output:
```{.console .no-copy}
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: http https ssh
  ports: 
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[root@localhost opt]#
```