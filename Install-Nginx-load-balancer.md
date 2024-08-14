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

Now we will make sure we are allowing the Nginx service to run through our firewall. The first command will enable the http server in the firewall.

```sh
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
```

Now, to make sure that the firewall rules are in effect, reload the firewall rules with the following command:

```sh
firewall-cmd --reload
firewall-cmd --list-all
```

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
In the example above, there are 3 instances of the same application running on 192.168.1.14, 192.168.1.15 & 192.168.1.16. When the load balancing method is not specifically configured, it defaults to round-robin. All requests are proxied to the server group myapp1, and nginx applies HTTP load balancing to distribute the requests.

### Least connected load balancing

Another load balancing discipline is least-connected. Least-connected allows controlling the load on application instances more fairly in a situation when some of the requests take longer to complete.

With the least-connected load balancing, nginx will try not to overload a busy application server with excessive requests, distributing the new requests to a less busy server instead.

Least-connected load balancing in nginx is activated when the least_conn directive is used as part of the server group configuration:

```sh
upstream myapp1 {
        least_conn;
        server 192.168.1.14;
        server 192.168.1.15;
        server 192.168.1.16;
    }
```

### Session persistence

Please note that with round-robin or least-connected load balancing, each subsequent client’s request can be potentially distributed to a different server. There is no guarantee that the same client will be always directed to the same server.

If there is the need to tie a client to a particular application server — in other words, make the client’s session “sticky” or “persistent” in terms of always trying to select a particular server — the ip-hash load balancing mechanism can be used.

With ip-hash, the client’s IP address is used as a hashing key to determine what server in a server group should be selected for the client’s requests. This method ensures that the requests from the same client will always be directed to the same server except when this server is unavailable.

To configure ip-hash load balancing, just add the ip_hash directive to the server (upstream) group configuration:

```sh
upstream myapp1 {
        ip_hash;
        server 192.168.1.14;
        server 192.168.1.15;
        server 192.168.1.16;
    }
```

### Weighted load balancing

It is also possible to influence nginx load balancing algorithms even further by using server weights.

In the examples above, the server weights are not configured which means that all specified servers are treated as equally qualified for a particular load balancing method.

With the round-robin in particular it also means a more or less equal distribution of requests across the servers — provided there are enough requests, and when the requests are processed in a uniform manner and completed fast enough.

When the weight parameter is specified for a server, the weight is accounted as part of the load balancing decision.

```sh
upstream myapp1 {
        server 192.168.1.14 weight=3;
        server 192.168.1.15;
        server 192.168.1.16;
    }
```

With this configuration, every 5 new requests will be distributed across the application instances as the following: 3 requests will be directed to 192.168.1.14, one request will go to 192.168.1.15, and another one — to 192.168.1.16.

It is similarly possible to use weights with the least-connected and ip-hash load balancing in the recent versions of nginx.
