# Nginx Setup:-

### Installation instructions

Before you install nginx for the first time on a new machine, you need to set up the nginx packages repository. Afterward, you can install and update nginx from the repository.

### RHEL and derivatives

This section applies to Red Hat Enterprise Linux and its derivatives such as CentOS, Oracle Linux, Rocky Linux, AlmaLinux.

```sh
sudo yum install yum-utils
```
To set up the yum repository, create the file named <b>/etc/yum.repos.d/nginx.repo</b> with the following contents:

```sh
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

By default, the repository for stable nginx packages is used. If you would like to use mainline nginx packages, run the following command:

```sh
sudo yum-config-manager --enable nginx-mainline
```

To install nginx, run the following command:

```sh
sudo yum install nginx
```

When prompted to accept the GPG key, verify that the fingerprint matches <b>573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62</b>, and if so, accept it.

### Load balancing methods

The following load balancing mechanisms (or methods) are supported in nginx:
- round-robin — requests to the application servers are distributed in a round-robin fashion,
- least-connected — next request is assigned to the server with the least number of active connections,
- ip-hash — a hash-function is used to determine what server should be selected for the next request (based on the client’s IP address).

### Default load balancing configuration

Please take note that in my situation, I removed all code from the default.conf file and added the load balancer rule listed below. But first, make a backup of the existing configuration file.

The simplest configuration for load balancing with nginx may look like the following:

```sh
upstream myapp1 {
        ip_hash;
        server 192.168.1.14;
        server 192.168.1.15;
        server 192.168.1.16;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
```

