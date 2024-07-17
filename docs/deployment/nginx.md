
# Installing & Configuring NGINX
The aim of this article is to get you started with basic Nginx web-server installation using the dnf 
install nginx command and configuration on RHEL 8 / CentOS 8. Nginx web server is an Apache alternative 
with a capability to be also used as reverse proxy, load balancer, mail proxy and HTTP cache.

---
## Procedures
* Install Nginx on RHEL 8 / CentOS 8.
* Run Nginx encrypted with HTTPS.
* Create a self-signed SSL certificate for Nginx.
---

## Install nginx on RHEL 7/8
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

* Open browser and navigate to `http://{{ server.ipaddress}}` URL.

Perform a further configuration of your host by editing the `/etc/nginx/nginx.conf` configuration file and server block: 

```console
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  _;
    root         /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```
!!! info
    The default Welcome to nginx web page location path is `/usr/share/nginx/html`.

## Enable HTTPS SSL support on Nginx and RHEL 8
Edit the /etc/nginx/nginx.conf configuration file and uncomment the entire settings for the TLS enabled server block:

```sh
sudo nano /etc/nginx/nginx.conf
```

```console title="Before"
server {
    listen       443 ssl http2 default_server;
    listen       [::]:443 ssl http2 default_server;
    server_name  _;
    root         /usr/share/nginx/html;

    ssl_certificate "/etc/pki/nginx/server.crt";
    ssl_certificate_key "/etc/pki/nginx/private/server.key";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

Replace the `location` block with the code below

```console
    location /stigman/ {
        proxy_pass              http://{{ server.ipaddress }}:54000/;
    }        
    location / {
        proxy_pass              http://{{ server.ipaddress }}:8080/;
        proxy_set_header        Host               $host;
        proxy_set_header        X-Real-IP          $remote_addr;
        proxy_set_header        X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header        X-Forwarded-Host   $host;
        proxy_set_header        X-Forwarded-Server $host;
        proxy_set_header        X-Forwarded-Port   $server_port;
        proxy_set_header        X-Forwarded-Proto  $scheme;
        proxy_set_header        ssl-client-cert    $ssl_client_escaped_cert;
        proxy_buffer_size       128k;
        proxy_buffers           4 256k;
        proxy_busy_buffers_size 256k;
    }
```

```console title="After"
    server {
        listen       443 ssl http2 default_server;
        listen       [::]:443 ssl http2 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/etc/pki/nginx/server.crt";
        ssl_certificate_key "/etc/pki/nginx/private/server.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers PROFILE=SYSTEM;
        ssl_prefer_server_ciphers on;
        
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location /stigman/ {
            proxy_pass              http://{{ server.ipaddress }}:54000/;
        }        
        location / {
            proxy_pass              http://{{ server.ipaddress }}:8080/;
            proxy_set_header        Host               $host;
            proxy_set_header        X-Real-IP          $remote_addr;
            proxy_set_header        X-Forwarded-For    $proxy_add_x_forwarded_for;
            proxy_set_header        X-Forwarded-Host   $host;
            proxy_set_header        X-Forwarded-Server $host;
            proxy_set_header        X-Forwarded-Port   $server_port;
            proxy_set_header        X-Forwarded-Proto  $scheme;
            proxy_set_header        ssl-client-cert    $ssl_client_escaped_cert;
            proxy_buffer_size       128k;
            proxy_buffers           4 256k;
            proxy_busy_buffers_size 256k;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

Create a directory to hold the SSL certificate and the private key for the Nginx server:
```sh
sudo mkdir -p /etc/pki/nginx/private/ 
```

Generate a self-signed certificate and private key or upload the existing one to 
the `/etc/pki/nginx/server.crt` and `/etc/pki/nginx/private/server.key` locations. 

The only required field when creating the self-signed certificate is `Common Name` (eg, your server's hostname):
```sh
openssl req -newkey rsa:2048 -nodes \
  -keyout /etc/pki/nginx/private/server.key \
  -x509 -days 3650 -out /etc/pki/nginx/server.crt
```

Open HTTPS port 443 on the firewalld firewall daemon:
```sh
firewall-cmd --zone=public --permanent --add-service=https
firewall-cmd --reload
```

Reload the Nginx configuration:
```sh
sudo systemctl reload nginx
```

Access the Nginx welcome page. All should now be ready to access Nginx from a remote host. 
Open the browser and paste the URL or navigate to [https://{{ server.ipaddress }}](https://{{ server.ipaddress }}).

## Next Step
[We will Configure Applications to use Proxy](deploy.md)