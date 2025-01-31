# Monitoring, Alerts and Scaling 

- [Monitoring, Alerts and Scaling](#monitoring-alerts-and-scaling)
    - [dashboard creation code along](#dashboard-creation-code-along)
      - [install apache bench:](#install-apache-bench)
  - [autoscaling](#autoscaling)
    - [why?](#why)
    - [types of scaling](#types-of-scaling)
    - [complext arcitecture involves planning](#complext-arcitecture-involves-planning)
    - [How does automated scaling work?](#how-does-automated-scaling-work)
    - [code-along - creating a virtual machine scale set](#code-along---creating-a-virtual-machine-scale-set)
      - [set up](#set-up)
      - [running the scale set](#running-the-scale-set)
  - [security](#security)
    - [securing the database using a 3 - subnet architecture](#securing-the-database-using-a-3---subnet-architecture)
      - [code-along set up new virtual network](#code-along-set-up-new-virtual-network)


### dashboard creation code along

* go to monitoring tab within overview 
* can look at metrics, which are running automatically 
* can create a custom dashboard: 
![pin to dash](../25.01.30/pin_to_dash.png)
*image missing? - add again*

#### install apache bench: 
sudo apt-get install apache2-utils
test cpu 

## autoscaling

### why? 
![cpu flow diagram](../25.01.30/fig2-why-scale.png)

### types of scaling 

| vertical | horisontal|
|---|---|
|scale up|scale out|

*research and add comparison table here*

### complext arcitecture involves planning 

* Azure VM scale set with high avalability (HA) and scalability 
* automatic scalability helps with high avalability, but they are not the same thing 

### How does automated scaling work? 

![scaling figure](../25.01.30/scaling_figure1.png)

### code-along - creating a virtual machine scale set 
#### set up 
* there are some different options to creating a regular virtual machine that need to be selected (if not specified use standard settings from week 1 notes)
* basics tab: 
  * tick avalability zones 1, 2 and 3 
  * orchestration mode: uniform 
  * scaling: autoscaling 
    * scaling configuration > edit condition 
      * ![scaling configuration screen](../25.01.30/scaling-config.png)
      * in summary: autoscale, 2,2,3
  * licencing: other
* network interface: 
  * use my virtual network (...subnet-2-...)
  * select public subnet 
  * advanced 
  * network security group: allow-http-ssh-3000
  * create a load balancer if needed or select own from dropdown - configure ports 50000 and 50001
  * ![create load balancer](<../Screenshot from 2025-01-30 14-59-29.png>)
* health tab: 
  * tick automatic repairs, leave rest of settings
* advanced tab: 
  * tick enable user data
  * insert bash script: 
  * 
    ```
    #!/bin/bash
    # navigating into app folder
    cd /repo/app
    #starting the app
    pm2 start app.js
    ```
#### running the scale set 

* if its running correctly the instances should both be running and passing
* when the vmss is relaunched you will need to re-image the instances in order to get them running 


## security 

### securing the database using a 3 - subnet architecture 
![diagram of subnet architecture](../25.01.30/vm-architecture-2.drawio.png)
* nic network interface card - connected to network security group which has rules about what can be allowed in 
* NIC enables interaction 
* NSG chooses what is allowed 
* port rules: 
  * public subnet: 
    * 80 http 
    * 22 ssh 
    * not port:3000 (open to internet)
  * private subnet (): 
    * 22 ssh 
    * 27017 mongo db (haven't had to use previously as azure default allows things in the same virtual network to communicate)
    * deny all other traffic 
  *ramon diagram red arrows = potentially dangerous traffic* 
  *database only has a public ip to allow us to ssh into it, otherwise not needed*
  *bastion host is an option, but very expensive*
  *delete public ip, shh in via public ip of app, then from there ssh imto db HOW??*
  *NVA - network virtual appliance* 
  *need to set up a route table*
  *NVA needs ip forwarding enabled in linux - have to log in* 
  *IP tables rules - other options avalable* 
  *only safe traffic is forwarded on to the db - green arrow (is filtered)

#### code-along set up new virtual network 

```
#!/bin/bash
# navigating into app folder
 cd /repo/app
 #connect to db
export DB_HOST=mongodb://10.0.4.4:27017/posts
 #starting the app
 pm2 start app.js 
```

enable in linux: 
sysctl net.ipv4.ip_forward
sudo nano /etc/sysctl.conf
sudo sysctl -p


pings should be back! 

script in nva - to configure iptables
```
#!/bin/bash
 
# configure iptables
 
echo "Configuring iptables..."
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -A INPUT -m state --state INVALID -j DROP
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
 
# uncomment the following lines if want allow SSH into NVA only through the public subnet (app VM as a jumpbox)
# this must be done once the NVA's public IP address is removed
#sudo iptables -A INPUT -p tcp -s 10.0.2.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
#sudo iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
 
# uncomment the following lines if want allow SSH to other servers using the NVA as a jumpbox
# if need to make outgoing SSH connections with other servers from NVA
#sudo iptables -A OUTPUT -p tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
#sudo iptables -A INPUT -p tcp --sport 22 -m conntrack --ctstate ESTABLISHED -j ACCEPT
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -A FORWARD -p tcp -s 10.0.2.0/24 -d 10.0.4.0/24 --destination-port 27017 -m tcp -j ACCEPT
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -A FORWARD -p icmp -s 10.0.2.0/24 -d 10.0.4.0/24 -m state --state NEW,ESTABLISHED -j ACCEPT
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -P INPUT DROP
 
# ADD COMMENT ABOUT WHAT THE FOLLOWING COMMAND(S) DO
sudo iptables -P FORWARD DROP
 
echo "Done!"
echo ""
 
# make iptables rules persistent
# it will ask for user input by default
 
echo "Make iptables rules persistent..."
sudo DEBIAN_FRONTEND=noninteractive apt install iptables-persistent -y
echo "Done!"
echo ""
``` 