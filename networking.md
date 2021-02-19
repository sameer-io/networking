#Networking basics

Working with Static Routes - we can set the routes specific to a nic as well 

```
ip route add 10.0.1.20 dev eth1
```
Send all traffic to that IP via eth1 interface - just in case if the eth1 is down then we cannot ping that IP as it's interface is down. 

Note: To down the interface use the command `ip link set eth1 down`

![image.png](./image.png)

The default route is the 1st line i.e all the traffic goes via 10.0.1.1
The other static routes in the bottom are explicitly specified i.e all the traffic to those IP should go via 10.0.1.20 


To remove an entry we can use `ip route flush ipaddr`

command to show existing routes: 
```
ip route show 
```


***
***
***

firewalld: - better than iptables - makes our lives easier 

once we install the firewalld, it comes with the pre-configured zones like deafult, public. block etc - behind the scenes the configs are nothing but xml files stating which services to allow - here services are like tcp, httpd, ssh and they are also configured in xml files which contanis the name and the port of that service 

Example: if the service httpd is allowed then we're instructing the OS to allow traffic over the port 80 which is the default port of that service

- similarly if we need ssh then then port 22 is activated 

![img](./image-1.png)

![image-2.png](./image-2.png)

![image-3.png](./image-3.png)

***

To understand the current status of the firewall rule we can use the command `firewall-cmd --list-all` 

![image-4.png](./image-4.png)

![image-5.png](./image-5.png)


In order to add specific port we can use the below command i.e we're permitting the port 100 

![image-6.png](./image-6.png)

![image-7.png](./image-7.png)

![image-8.png](./image-8.png)

***

We can also create owr own custom services i.e we can create an xml entry or we can we inline command as well 

![image-9.png](./image-9.png)

***

we created a service and next step is to create an IPsets - in the commannds mentioned below our objective is to create a custom zone and a service called jmx along with an ipset - our main aim to make sure that the ip's mentioned on the ipset will only be able to access the jmx service - apart from that service they shouldn't be able to access any other services. 

So what we did is simple - we created a zone and binded the jmx srvice to that zone and similary we binded the ipsets to that zone now as soon as we switched to that new zone - our objective is achieved. 

***

##RICH RULES: are used to control the rules at very granual level 

In the below commands we've created a rich rule - let us assume our subnet is 10.0.1.0/24 and in this subnet we have a specific host which is running a service - let us assume that specifc host's IP is 10.0.1.10/24 - our objective is to accept all the traffic from our subnet to be able to access this service - we add a rich rule stating that "permit all the ips from the source address range to be able to access a specific range of port in the specific destination host i.e 10.0.1.10/24" - now all the ips in the subnet will be able to access the srvice running in the destination host. 



![image-11.png](./image-11.png)





![image-10.png](./image-10.png)


***

#TROUBLESHOOTING FIREWALL 

Reference architecture. 

![image-12.png](./image-12.png)

The below screenshot is to verify what services are running and their corresponding ports - we verified that the local service is running on port 80 

In this troubleshooting section we're fixing the connection b/w client and the server 

- client IP: 10.0.1.11
- Server IP: 10.0.1.10 

Now when we curl the server from client machine we're getting error as shown below: 
![image-13.png](./image-13.png)
![image-14.png](./image-14.png)

Now as we're unable to curl the server from our client machine - we've logged in to our server to see the ip table rules using the command `iptables -vnL` - remember that it's all about the order of rules in the iptables - the order is very very very importanat - as you can see the last line in the INPUT chain rules i acepting the 80 port traffic but if you see the line just above that we're rejecting everything. 

while evaluating the rules the evaluation process starts from the top row and it keeps going untill it founds a match - as soon as the reject rule is reached it mtaches and it stops travesing the table. 




![image-15.png](./image-15.png)


#Introduction
In this hands-on lab, you will need to create a script that modifies the routing table to prohibit connectivity to google.com. You'll also add an entry to use 10.0.1.20 as the gateway for the 10.0.8.0/24 subnet.

Solution
Begin by logging in to the lab server using the credentials provided on the hands-on lab page:

ssh cloud_user@PUBLIC_IP_ADDRESS

Become the root user:

sudo su -

View the routing table, and prohibit connectivity to google.com.
You can view the routing table using the following command:
```
ip route show
```
You should install the bind-utils package which includes the host command, to get the IPs associated with google.com.
```
host google.com
```
Once you have the IP address(es), you can prohibit connectivity to them using the following command:
```
ip route add prohibit <IP_ADDRESS>
```
Add an entry for the 10.0.8.0/24 network to use 10.0.1.20 as the gateway
Add the following entry:
```
ip route add 10.0.8.0/24 via 10.0.1.20
```
Write a script that creates both entries
Using a text editor like vim, create and edit the file /home/cloud_user/routes.sh :

```
#!/bin/bash
	 
ip route add prohibit <GOOGLE_IP>
ip route add 10.0.8.0/24 via 10.0.1.20
```
Conclusion
Congratulations — you've completed this hands-on lab!

***
***
***
***
***
***
###Port Forwarding with the Firewall

The Scenario
A business unit is requesting the ability to serve content from an in-development web stack to a subnet, in order to facilitate validation and testing.

We have three hosts:

Server1 10.0.1.10: The current web server
Server2 10.0.1.20: The in-development web server
Client1 10.0.1.11: For testing

We need to configure Server1 so that incoming web traffic (port 80) requests from 10.0.1.0/24 are forwarded to Server2. Requests from all other sources should remain unforwarded. We will need to do this using firewalld.

Logging In
Use the login information on the hands-on lab overview page to access the servers with SSH. Pay attention to the shell prompts in the lab guide, as they'll indicate which server we're in when we run a command.

##Verify port 80 is open on Server1 and Server2

We'll want to verify that content is being served over port 80 on both Server1 and Server2.

From Client1
We can verify web content availability from Server1 and Server2 with these commands:

```
[cloud_user@Client1]$ curl 10.0.1.10
[cloud_user@Client1]$ curl 10.0.1.20

```

Create a Zone, testing, to Handle the Subnet Requests
On Server1
Once we're logged in, become root (with the su - command).

##Create a new firewall zone and reload the configuration, to pick up the new zone:

```
[root@Server1]# firewall-cmd --permanent --new-zone=testing
[root@Server1]# firewall-cmd --reload
```
##Add the subnet as the source:
```
[root@Server1]# firewall-cmd --permanent --zone=testing --add-source=10.0.1.0/24`
```
Make sure http as a service is added and reload the configuration again to pick up these changes:
```
[root@Server1]# firewall-cmd --permanent --zone=testing --add-service=http`
[root@Server1]# firewall-cmd --reload
```
##Enable Masquerading for the Zone
We need to enable masquerading for the zone, in order to permit forwarding, and reload again:

```
[root@Server1]# firewall-cmd --permanent --zone=testing --add-masquerade
[root@Server1]# firewall-cmd --reload
```

##Add the Forwarding Rule to the Zone
We have to add a rule that forwards traffic coming in from the testing zone on port 80 out to 10.0.1.20:80, and reload the configuration again:
```
[root@Server1]# firewall-cmd --permanent --zone=testing  \
--add-forward-port=port=80:proto=tcp:toport=80:toaddr=10.0.1.20`
[root@Server1]# firewall-cmd --reload
```
Confirm the Port is Forwarded
We can confirm that the port is forwarded by running curl on the site from Client1:
```
[cloud_user@Client1]# curl 10.0.1.10
```
We see web content from Server2. To test further, we can run curl on the public IP of Server1, and we'll see Server1 web content instead. That's good, because we only wanted to forward traffic coming from the specified subnet.

##Conclusion

Here's the rundown of what we've done. We confirmed that Client1 can access the web servers on Server1 and Server2 directly. In the firewall on Server1, we added a new zone called testing. Then we added the subnet Client1 is sitting in as a source for zone testing, opened port 80, enabled masquerading. Finally, we added a "forward" port, so any traffic coming from the specified subnet on port 80 is forwarded from Server1 to Server2.

That's quite a job. Congratulations!
