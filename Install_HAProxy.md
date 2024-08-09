# HAProxy Setup:-

## Loadbalancer Server

#### Step 1: Install HAProxy on CentOS 7 Load Balancing Server:-

```sh
yum install haproxy
```
Once the installation finishes, start the HAProxy service with the following command:

```sh
systemctl start haproxy
systemctl enable haproxy
systemctl status haproxy
chkconfig haproxy on
```

Now we will make sure we are allowing the HAProxy service to run through our firewall. The first command will enable the http server in the firewall.

```sh
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
```

Now, to make sure that the firewall rules are in effect, reload the firewall rules with the following command:

```sh
firewall-cmd --reload
firewall-cmd --list-all
```

#### Step 2: Install the Web Server on the Backend Servers:-

The other two servers would act as the backend servers. For this, we’ll set up Apache as the webserver to service requests that the HAProxy server would redirect.

We chose Apache because it’s perhaps the most widely used web server today. It is open source and has been a popular choice for setting up servers for web apps.

# Now on App servers 1 & 2, use the following commands to install Apache:


## App Server 1 & 2

```sh
# yum install httpd
```

Once the installation finishes, you’ll notice that the Apache service doesn’t start automatically. You need to enter the following command to start the service.

```sh
systemctl start httpd
systemctl enable httpd
```

#### Step 3: Configure Firewall Settings on Apache Servers

```sh
firewall-cmd --permanent --zone=public --add-service=http
firewall-cmd --permanent --zone=public --add-service=https
```

Now, to make sure that the firewall rules are in effect, reload the firewall rules with the following command:

```sh
firewall-cmd --reload
firewall-cmd --list-all
```

## Now on Loadbalancer Server

#### Step 4: Add Backend Servers to the HAProxy Configuration File or configure haproxy in Centos 7

At this point, we have installed HAProxy and Apache servers on the test servers.

To connect these components, we need to make changes to the haproxy.cfg (the HAProxy configuration file). You can see that we’ve commented out the existing content of the file and added our rules.

Start by entering the following command in the terminal of the HAProxy server.

```sh
vi /etc/haproxy/haproxy.cfg
```

Now need add the following rule to the file at last.

```sh
############ Configure Frontend Server ###############
frontend webapp
        bind *:80
        stats enable
        stats auth admin:123
        stats uri /stats
        stats refresh 10s
        stats admin if LOCALHOST
        default_backend web-servers


############### Configure Backend server ################
backend web-servers
        balance roundrobin
        server app1 192.168.1.11:80 check
        server app2 192.168.1.12:80 check
```

Save the file and restart the HAProxy service to ensure the new rules are loaded and in use.

```sh
systemctl restart haproxy
```
You should note that you can use other custom ports instead of the default ports 80 and 443. Another essential fact to remember is that the default algorithm is Round Robin.

Also, note that the check option (at the end of the server directives) forces health checks before the backend server is used.

## HAProxy Status Dashboard

HAProxy also offers a dashboard called the HAProxy Status page. Here, you can see the essential metrics that present server health, current request rates, response times, and more.

You must enable a directive in the HAproxy configuration file to add a metric. For instance, in the following screenshot, you can see that you need to provide credentials to access the Status page. The authentication directive we added to the HAProxy config file requires users to enter credentials before accessing the page.

```sh
http://192.168.1.3/stats
```

For HAProxy dashboard we have added rule on file: haproxy.cfg.

```sh
stats enable
stats auth admin:123
stats uri /stats
stats refresh 10s
stats admin if LOCALHOST
```

Once you provide the proper credentials, you can access the HAProxy Status page to access vital statistics about the load balancer.
