
# Installing & Configuring NGINX
The aim of this section is to get you started with basic Nginx web-server installation using the dnf 
install nginx command and configuration on RHEL 8 / CentOS 8. Nginx web server is an Apache alternative 
with a capability to be also used as reverse proxy, load balancer, mail proxy and HTTP cache.

---
## Steps
1. [Install Nginx](#install-nginx)
2. [Enable HTTPS SSL support on Nginx](#enable-https-ssl-support-on-nginx)
3. [Create PKI directory for Keys and Certificates](#create-pki-directory-for-keys-and-certificates)
4. [Generate SSL CA Keys and Certificates](#generate-ssl-ca-keys-and-certificates)
5. [Generate SSL Server Keys and Certificates](#generate-ssl-server-keys-and-certificates)
6. [Open HTTPS and HTTP ports (443 and 80)](#open-https-and-http-ports-443-and-80)

---
## Install Nginx
Install package nginx using the dnf command.
```sh
sudo dnf install nginx
```

Start the Nginx service:
```sh
sudo systemctl start nginx
```

To ensure that Nginx starts after the reboot enable systemd service the nginx:
```sh
sudo systemctl enable nginx
```

Execute the below commands to check the status of the service:
``` bash
sudo systemctl status nginx.service
```

Output:
```console
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
```

Open HTTP firewall port 80:
```sh
sudo firewall-cmd --zone=public --permanent --add-service=http
sudo firewall-cmd --reload
```

Backup the nginx configuration file `/etc/nginx/nginx.conf`
```sh
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.distro
```

!!! info
    All should now be ready to access Nginx from a remote host. 

Access the Nginx welcome page.

* Open browser and navigate to `{{ server.web }}` URL.

---
## Enable HTTPS SSL support on Nginx
Copy the nginx sever configuration file to `/etc/nginx/conf.d`

Execute the below commands to copy the config file:
``` bash
sudo cp /opt/deployment/nginx/etc/nginx/conf.d/stigman.nginx.conf /etc/nginx/conf.d
```

[View this file as plain text](../_downloads/stigman.nginix.txt)

```console
# generated 2024-06-05, Mozilla Guideline v5.7

server {
    listen                  443 ssl http2;
    listen                  [::]:443 ssl http2;
    server_name             eh-stigman-1;
    ssl_certificate         /etc/pki/nginx/eh-stigman-1.crt;        
    ssl_certificate_key     /etc/pki/nginx/private/eh-stigman-1.key;     

    ssl_session_timeout     1d;
    ssl_session_cache       shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets     off;

    # curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam
    # ssl_dhparam             /etc/pki/nginx/dhparam;

    # intermediate configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;


    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    # ssl_trusted_certificate /etc/pki/nginx/ca-chain.cert.pem;
 

    location / {
        proxy_pass              http://eh-stigman-1:54000/;
    }

    location /auth/ {
        proxy_pass              http://eh-stigman-1:8080/auth/;
        proxy_set_header        Host               $host;
        proxy_set_header        X-Real-IP          $remote_addr;
        proxy_set_header        X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Host   $host;
        proxy_set_header        X-Forwarded-Server $host;
        proxy_set_header        X-Forwarded-Port   $server_port;
        proxy_set_header        X-Forwarded-Proto  X$scheme;
        proxy_set_header        ssl-client-cert    $ssl_client_escaped_cert;
        proxy_buffer_size       128k;
        proxy_buffers           4 256k;
        proxy_busy_buffers_size 256k;
    }
}

```

## Create PKI directory for Keys and Certificates
Create a directory to hold the SSL certificate and the private key for the Nginx server.

!!! info
    The cert will be created and placed in `/etc/pki/nginx/server.crt`

    the key will be created and placed in `/etc/pki/nginx/private/server.key` 

Execute the below commands to create the directories:
```sh
sudo mkdir -p /etc/pki/nginx/private/ 
```

```sh
sudo mkdir -p /opt/pki
```

```sh
cd /opt/pki
```

## Generate SSL CA Keys and Certificates

!!! note
    Skip the two step below if you already have a a signing certificate step if you 

Execute the below command to generate CA's private key and self-signed certificate
``` sh

  openssl req -x509 -newkey rsa:4096 \
    -days 3650 \
    -nodes \
    -keyout ca-key.pem \
    -out ca-cert.pem \
    -subj "/C=US/ST=Connecticut/L=East Hartford/O=Alice Ltd/OU=Finance/CN=Alice Ltd Certificate Authority/emailAddress=alice.ca@support.com"
```

Execute the below command to  create the CA's self-signed certificate:
``` sh
openssl x509 -in ca-cert.pem -noout -text
```


## Generate SSL Server Keys and Certificates

Execute the below command to generate web server's private key and certificate signing request (CSR)
``` sh
openssl req -newkey rsa:4096 \
    -keyout server-key.pem \
    -out server-req.pem \
    -subj "/C=US/ST=Connecticut/L=East Hartford/O=Alice Ltd/OU=Finance/CN=*.staging.local/emailAddress=alice.finance@support.com"
```

Execute the below command to create the config file:

!!! note
    Edit the DNS, IP, and wildcard information below before running the command.


``` sh
echo "subjectAltName=DNS:*.monofinance.net,DNS:*.monofinance.com,DNS:*.monofinance.org,IP:0.0.0.0" \
 > server-ext.cnf
```

Execute the below command to use CA's private key to sign web server's CSR and get back the signed certificate
``` sh
openssl x509 -req -in server-req.pem \
    -CA ca-cert.pem \
    -CAkey ca-key.pem \
    -CAcreateserial \
    -out server-cert.pem -days 3650 \
    -extfile server-ext.cnf
```

Execute the below command below to show the Server's signed certificate:
``` sh
openssl x509 -in server-cert.pem -noout -text
```

Execute the below command below to verify Server and CA certificates:
``` sh
openssl verify -CAfile ca-cert.pem server-cert.pem
```

Execute the below commands below to copy the file to the appropriate directories:
``` sh
cp server-cert.pem /etc/pki/nginx/server.crt
cp server-cert.key  /etc/pki/nginx/private/server.key
```

Reload the Nginx configuration to pick load the new certs:
```sh
sudo systemctl restart nginx.service
```

---
## Open HTTPS and HTTP ports (443 and 80)

Execute the below command below to open the port:
``` sh
firewall-cmd --zone=public --permanent --add-service=https
firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --reload
```

Access the Nginx welcome page. All should now be ready to access Nginx from a remote host. 
Open the browser and paste the URL or navigate to [{{ server.ssl }}]({{ server.ssl }}).


---
## Summary
!!! success annotate "Summary"
    In these procedures, on a RHEL 7/8 distributions, you have:

    * Installed and enabled Nginx
    * Enabled HTTPS SSL support on Nginx
    * Created PKI directory for Keys and Certificates
    * Generated SSL CA Keys and Certificates
    * Generated SSL Server Keys and Certificates
    * Opened HTTPS and HTTP ports (443 and 80)