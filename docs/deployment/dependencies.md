# Dependencies

## Steps
1. [Installing Openjdk Using dnf](#installing-openjdk-using-dnf)
2. [Open Ports for STIG Manager & keycloak (Temporary)](#open-ports-for-stig-manager-and-keycloak-temporary)


----
## Installing Openjdk Using dnf
You can install Red Hat build of OpenJDK Java Runtime Environment (JRE) using the system package manager, dnf.

Run the yum command, specifying the package you want to install:
```sh
sudo dnf install java-17-openjdk
```

Check that the installation works:
```sh
java -version
```

Output:
```sh
openjdk version "17.0.11" 2024-04-16 LTS
OpenJDK Runtime Environment (Red_Hat-17.0.11.0.9-3) (build 17.0.11+9-LTS)
OpenJDK 64-Bit Server VM (Red_Hat-17.0.11.0.9-3) (build 17.0.11+9-LTS, mixed mode, sharing)
```

!!! note
    If the output from the previous command shows that you have a different major version of Red Hat build of OpenJDK checked out on your system, you can enter the following command in your CLI to switch your system to use Red Hat build of OpenJDK 17:


```sh
sudo update-alternatives --config 'java'
```

```sh
There is 1 program that provides 'java'.

  Selection    Command
-----------------------------------------------
*+ 1           java-17-openjdk.x86_64 (/usr/lib/jvm/java-17-openjdk-17.0.11.0.9-2.el8.x86_64/bin/java)

Enter to keep the current selection[+], or type selection number:
```


## Open Ports for STIG Manager and keycloak (Temporary)
!!! note
    If the firewall is not running skip this step.

    The `--permanent` flag is not used in these commands. Settings will go back to default after reboot.

By default, the port 443, 8080 & 54000 are filtered on Redhat 7 and 8 as you can only access this port from the actual localhost and not from any other public host. To open a port 80 on RHEL 7 and 8 Linux we need to add an iptables rule. For this RHEL uses `firewall-cmd`.


Execute the below commands to to check the firewall state.
```sh
firewall-cmd --state
```

Output:
```{.console .no-copy}
running
[root@localhost opt]#
```

Execute the below commands to add port 8080 rule for keycloak: 
```sh
firewall-cmd --zone=public --add-port=8080/tcp --permanent
```

Execute the below commands to add port 54000 rule for stig-manager: 
```sh
firewall-cmd --zone=public --add-port=54000/tcp --permanent
```

Execute the below commands to reload the firewall service: 
```sh
firewall-cmd --reload
```

Execute the below commands to check  for open ports and services.
```sh
firewall-cmd --list-all
```

!!! info
    Open ports are listed on line starting with ports:

    The `--permanent` flag is used in this commands. Settings will persist after reboot.

Output:
```{.console .no-copy}
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens160
  sources:
  services: ssh
  ports: 54000/tcp 8080/tcp
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

## Summary
!!! success annotate "Summary"
    In these section, you have:

    * Installed openjdk using dnf
    * Opened ports Temporary for stig-manager & keycloak

