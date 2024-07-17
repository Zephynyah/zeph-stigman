# Installing & Configuring STIG Manager

!!! info
    STIG Manager binaries are made available [with each release.](https://github.com/NUWCDIVNPT/stig-manager/releases)

## Steps
1. [Extract STIG Manager Server](#extract-stig-manager-application)
2. [Create a systemd Unit File for stigman](#creating-a-systemd-unit-file-for-stigman)
3. [Create a symbolic link (symlink) folder](#create-a-symbolic-link-symlink-folder)
4. [Enable SIG Manager service on system startup](#enable-stigman-service-on-system-startup)
5. [Configure STIG Manager](#configure-stig-manager)

## Extract STIG Manager Application
We are going to install STIG Manager to `/opt/` directory, so we will extract the 
STIG Manager package to that location. 

Execute the below commands to move to the `/opt` directory:
```sh
cd /opt/
```

Execute the below commands to unzip the STIG Manager:
```sh
sudo tar -xf /opt/deployment/stigman/stig-manager-linux-1.4.11.tar.xz \
--one-top-level="stig-manager-1.4.11" \
--strip-components=1
```

---
## Creating a systemd Unit File for stigman
Copy systemd unit file (`stigman.service`) under `/opt/deployment/stigman/scripts/systemd/` to `/etc/systemd/system/` directory.

!!! info
    More information bout system D files can be found here. [systemd Unit Files](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/using_systemd_unit_files_to_customize_and_optimize_your_system/assembly_working-with-systemd-unit-files_working-with-systemd#ref_important-service-section-options_assembly_working-with-systemd-unit-files)

Execute the below commands to move to `../systemd/` deployment directory:
``` sh
cd /opt/deployment/stigman/scripts/systemd/
```

Execute the below commands to copy the file:
``` sh
sudo cp ./stigman.service /etc/systemd/system/stigman.service
```

Execute the below commands to view contents of the file. Exit the editor when done.
``` sh
nano /etc/systemd/system/stigman.service
```

Below is the content of the `stigman.service` file
``` sh
[Unit]
Description=The STIG Manager Application Server
After=syslog.target network.target
Before=httpd.service

[Service]
Type=simple
LimitNOFILE=102642
AmbientCapabilities=CAP_SYS_ADMIN
PIDFile=/var/run/stigman/stigman.pid
WorkingDirectory=/opt/stig-manager
ExecStart=/bin/bash -c "./stig-manager.sh"

# Logging
StandardOutput=append:/var/log/stig-manager.log
StandardError=append:/var/log/stig-manager.log

SuccessExitStatus=0 143
Restart=always
RestartSec=60
TimeoutStartSec=60
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target
```

Execute the below commands to set the proper permissions over a unit file:
``` sh
chown root:root /etc/systemd/system/stigman.service
chmod 0644 /etc/systemd/system/stigman.service
```

---
## Create a symbolic link (symlink) folder.
A symlink is used hear to make the stig-manager-{{ stigman.version }}  folder appear in the same directory on the 
filesystem but with different names.

!!! info
    This is used to simply version changes without changing the main configuration files. 
    
    For example: the systemd file "`stigman.service`", will always point to stig-manager rather that a specific version.

Execute the below commands to create the symlink for stig-manager:

``` sh
sudo ln -s /opt/stig-manager-{{ stigman.version }} /opt/stig-manager
```

!!! note
    Below is an example of what this directory would look like for an upgrade. 
    
    To remove the `sudo unlink /opt/stig-manager`

    To link `stigman-{{ stigman.upgrade }}` run `sudo ln -s /opt/stigman-{{ stigman.upgrade }} /opt/stig-manager`
```
```sh
...
lrwxrwxrwx. 1 root     root       21 Jul  6 01:45 keycloak -> /opt/keycloak-24.0.4/
drwxr-xr-x. 8 keycloak keycloak  138 Jul  6 11:00 keycloak-24.0.4
drwxr-xr-x. 8 root     root      163 Jul  6 02:12 keycloak-24.0.5
...
lrwxrwxrwx. 1 stigman  stigman    24 Jul  7 16:58 stig-manager -> /opt/stig-manager-1.4.11
drwxr-xr-x. 2 root     root       61 Jul  7 16:55 stig-manager-1.4.11
...
```

---
## Enable stigman service on system startup
Reload systemd manager configuration 
``` sh
sudo systemctl daemon-reload
```

Execute the below commands to start the service:
``` sh
sudo systemctl start stigman
```

Execute the below commands to check the status of the service:
``` sh
sudo systemctl status stigman
```

Execute the below commands to enable the service on system startup:
``` sh
sudo systemctl enable stigman
```

Execute the below commands to look at the logs
```sh
tail -f /var/log/stig-manager.log
```

Execute the below commands to stop the service:
```sh
sudo systemctl stop stigman
```

---
## Configure STIG Manager 
Reload systemd manager configuration 

Execute the below commands to edit stig-manager environment variables file.
```sh
sudo nano /opt/stig-manager-{{ stigman.version }}/stig-manager.sh
```

``` sh title=" stig-manager.sh Before"
# export STIGMAN_CLASSIFICATION=
...
# export STIGMAN_DB_PASSWORD=
...
# export STIGMAN_CLIENT_OIDC_PROVIDER=
...
# export STIGMAN_OIDC_PROVIDER=
```

Un-comment the following variable and update their values:
``` sh title="stig-manager.sh After"
export STIGMAN_CLASSIFICATION=C
...
export STIGMAN_DB_PASSWORD=Password123!
...
export STIGMAN_CLIENT_OIDC_PROVIDER=http://{{ stigman.ipaddress }}/realms/stigman
...
export STIGMAN_OIDC_PROVIDER=http://localhost:8080/realms/stigman
```


Execute the below commands to restart service:
``` sh
sudo systemctl restart stigman.service
```

## Summary
!!! success annotate "Summary"
    In these procedures, on a RHEL 7/8 distributions, you have:

    * Extracted STIG Manager Server
    * Created a systemd Unit File for stigman
    * Created a symbolic link (symlink) folder.
    * Enabled SIG Manager service on system startup
    * Configured STIG Manager
