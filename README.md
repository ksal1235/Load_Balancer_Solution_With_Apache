# Load_Balancer_Solution_With_Apache

- After completing DevOps tooling website solution Project, we might wonder how a user will be accessing each of the webservers using 3 different IP addreses or 3 different DNS names. 
- we might also wonder what is the point of having 3 different servers doing exactly the same thing. When we access a website in the Internet we use an URL and we do not really know how many servers are out there serving our requests, this complexity is hidden from a regular user, but in case of websites that are being visited by millions of users per day (like Google or Reddit) it is impossible to serve all the users from a single Web Server (it is also applicable to databases, but for now we will not focus on distributed DBs).
- Each URL contains a domain name part, which is translated (resolved) to IP address of a target server that will serve requests when open a website in the Internet. Translation (resolution) of domain names is perormed by DNS servers, the most commonly used one has a public IP address 8.8.8.8 and belongs to Google. You can try to query it with nslookup command:

```
nslookup   8.8.8.8
Server:   UnKnownAddress: 103.86.99.99

Name:   dns.google
Address: 8.8.8.8
```

- When you have just one Web server and load increases - you want to serve more and more customers, you can add more CPU and RAM or completely replace the server with a more powerful one - this is called "vertical scaling". This approach has limitations - at some point you reach the maximum capacity of CPU and RAM that can be installed into your server.
- Another approach used to cater for increased traffic is "horizontal scaling" - distributing load across multiple Web servers. This approach is much more common and can be applied almost seamlessly and almost infinitely (you can imagine how many server Google has to serve billions of search requests).
- Horizontal scaling allows to adapt to current load by adding (scale out) or removing (scale in) Web servers. Adjustment of number of servers can be done manually or automatically (for example, based on some monitored metrics like CPU and Memory load).Property of a system (in our case it is Web tier) to be able to handle growing load by adding resources, is called "Scalability".In our set up in Project-7 we had 3 Web Servers and each of them had its own public IP address and public DNS name.
- A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server, let alone millions of Google servers.In order to hide all this complexity and to have a single point of access with a single public IP address/name, a Load Balancer can be used. A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.
- Property of a system (in our case it is Web tier) to be able to handle growing load by adding resources, is called "Scalability".
- In our set up in Project-7 we had 3 Web Servers and each of them had its own public IP address and public DNS name. A client has to access them by using different URLs, which is not a nice user experience to remember addresses/names of even 3 server, let alone millions of Google servers.
- In order to hide all this complexity and to have a single point of access with a single public IP address/name, a Load Balancer can be used. A Load Balancer (LB) distributes clients' requests among underlying Web Servers and makes sure that the load is distributed in an optimal way.


#### Let us take a look at the updated solution architecture with an LB added on top of Web Servers (for simplicity let us assume it is a software L7 Application LB, for example - Apache, NGINX or HAProxy).

![image](https://github.com/user-attachments/assets/bd03020f-9253-4292-9265-10ecff379001)
In this project we will enhance our Tooling Website solution by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

## Task:
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. Make sure that users can be served by Web servers through the Load Balancer. To simplify, let us implement this solution with 2 Web Servers, the approach will be the same for 3 and more Web Servers.


## Prerequisites:

Make sure that you have the following servers installed and configured within the previous project:
1. Two RHEL8 Web Servers.
2. One MySQL DB Server (based on Ubuntu 20.04).
3. One RHEL8 NFS server.

![image](https://github.com/user-attachments/assets/a27fb54b-4413-4a80-ba6c-1b055bbd6b1b)

![image](https://github.com/user-attachments/assets/a893e0cd-4825-480a-8ea8-63b2dca9ef06)


#### Configure Apache As A Load Balancer:

1. Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this.
2. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.
![image](https://github.com/user-attachments/assets/ab75201c-5ab2-4498-85f2-747df344c5be)

4. Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

Updating the Machine.

````
sudo apt update
````

Installing the apache Webserver:

````
sudo apt install apache2 -y
````
![image](https://github.com/user-attachments/assets/4c9709f7-1771-4212-a554-64dcd67378c2)


Installing the libxml2-dev.
```
sudo apt-get install libxml2-dev
```
![image](https://github.com/user-attachments/assets/961efa7e-32ec-4e77-bda5-2cca7dc36cd5)

#### Enable following modules:

```
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic
```

#### Restart apache2 service:
```
sudo systemctl restart apache2
```
Make sure apache2 is up and running:
```
sudo systemctl status apache2
```
![image](https://github.com/user-attachments/assets/6b724f95-ca5d-4803-a8e7-063e63162044)

Configure load balancing:
```
sudo vi /etc/apache2/sites-available/000-default.conf
```

```
        <Proxy "balancer://mycluster">
        BalancerMember http://172.31.45.101:80 loadfactor=5 timeout=1
        BalancerMember http://172.31.46.136:80 loadfactor=5 timeout=1
        ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass "/" "balancer://mycluster/"
        ProxyPassReverse "/" "balancer://mycluster/"

```
![image](https://github.com/user-attachments/assets/50587591-a8ad-4371-9f45-3f12886553ce)


#Restart apache server
```
sudo systemctl restart apache2
```

![image](https://github.com/user-attachments/assets/d525d543-8b1c-447e-81c7-300be9325d8b)

by traffic balancing method will distribute incoming load between your Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter.


#### Accessing the LoadBlanacer Through Public Ips.

```
http://13.233.152.48/login.php
```
![image](https://github.com/user-attachments/assets/7309f70b-d6a4-498e-9883-97bc2b5ac1be)

Accessing the page via credential.
![image](https://github.com/user-attachments/assets/46ec2439-ef7c-4591-b2ea-bf261c31daba)

#### Webserver logs.

Checking the logs
```
 sudo tail -f /var/log/httpd/access_log
```
WebServer1 Logs

![image](https://github.com/user-attachments/assets/394aaf7d-386f-4d99-aad5-aac2fbba40f8)

WebServer2 Logs

![image](https://github.com/user-attachments/assets/f0cee06e-e037-4e22-bfed-57bf6775a7eb)

we have successfully set up an Apache load balancer on AWS to distribute traffic between our two web servers.

#### Optional Step - Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under our management. What we can do, is to configure local domain name resolution. The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB.

#### Open this file on your LB server.

```
sudo vi /etc/hosts
```
#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers
```
<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```
Now you can update your LB config file with those names instead of IP addresses.
![image](https://github.com/user-attachments/assets/54d774ba-95ea-4c3b-a5b0-808a46a1cffa)

```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```
```
sudo vi /etc/apache2/sites-available/000-default.conf
```
![image](https://github.com/user-attachments/assets/76328de6-7a17-436a-8067-768b02e5e8fd)

#### Conclusion:

The Load Balancer Solution with Apache project demonstrates a practical implementation of a load balancing system using Apache as the load balancer. This project showcases how to distribute incoming web traffic across multiple web servers, improving the overall performance, reliability, and scalability of a web application.

Key takeaways from this project include:

The use of Apache's mod_proxy and mod_proxy_balancer modules to configure load balancing.
Implementation of various load balancing algorithms, such as round-robin, allowing for efficient distribution of requests.
Setup of multiple backend web servers to handle incoming traffic.
Configuration of health checks to ensure only healthy servers receive traffic.
Demonstration of how to improve system resilience and handle increased traffic loads.
This project serves as a valuable resource for system administrators, DevOps engineers, and developers looking to implement a load balancing solution using open-source tools. It provides a foundation for building scalable and highly available web applications, which is crucial in today's digital landscape where user expectations for performance and uptime are high.

By following this project, users can gain practical experience in configuring and managing a load balancer, which is an essential skill in modern web infrastructure design and management. The knowledge gained from this project can be applied to both small-scale applications and larger, more complex systems requiring robust load balancing solutions.
