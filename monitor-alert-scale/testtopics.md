# Consultancy questions about: 
* how to foster a DevOps culture 
* usual/typical meetings for a DevOps Engineer 
* things to do that will help you in your first week as a Junior DevOps Engineer 
* who a DevOps Engineer would usually need to interact with for BAU 
* BAU tasks for a DevOps Engineer 
 # Technical questions about: 
## 2-tier architecture deployment of Sparta test app 
* why put app and database in separate subnets 
  ```
  enhances security by isolating network traffic, allows better control of what resources have access to each subnet
  ``` 
* what to check to make sure your database provision script did it's job 
  ``` 
  check script after and make sure it has seeded the database after running npm install in the app folder. Also check the localhost/posts page to see if the data is there
  ``` 
* NSG rule to allow Mongo DB traffic 
   * addition of network security group rule with the following fields: 

    | Field	| Value | 
    |---| --- |
    |Source	|Any or specific IP/Virtual Network|
    |Source Port Range	|*|
    |Destination|	Any or specific MongoDB server IP|
    |Destination Port Range|27017|
    |Protocol|TCP|
    |Action | Allow|
    |Priority|	100 |
    |Name |	Allow-MongoDB-Traffic |


* steps needed to get /posts page working 
* ways to run the app as a background process 
```
pm2 manager or & in usual run command 
``` 
* how to stop the app running after running the app with the pm2 command in user data (tricky! Try it out if you're not sure) 
* understand bindIp, what setting to use for testing vs production 
  ```
  The bindIp setting refers to the IP address or addresses that a service (such as a database, web server, or application) will listen on for incoming network connections.

  It's common to bind to local host whilst testing (127.0.0.1 or ::1) to prevent accidental exposure to an outside network or bind to a specific test network 

  depends who needs to be able to access your service, may bind to an ip for the internal network or bind all network interfaces (0:0:0:0) if it needs to be accessable to all 

  can also bind to a production specific range of ip's in the case of cloud deployment using load balancers or reverse proxys to handel routing 
  ``` 
## understanding how user data works 
```
User data in Azure refers to a set of commands, scripts, or configuration data that you pass to a VM instance during creation, allowing you to automate the setup or customization of the machine on its first boot (so only runs once!) 

The most common use cases include:

    Installing software or packages.
    Running configuration scripts (e.g., Bash scripts for Linux or PowerShell scripts for Windows).
    Setting up specific environment variables or services
  ``` 
## familiar with the process to create a generalised image 
```
`waagent -deprovision+user` in the VM 
then in azure - stop the vm, dealocate, click 'capture > image' 
``` 
## process/methods to automate deployment of an app on the cloud 
## how to automate configuration of the Nginx reverse proxy 
## VM Scale Sets 
* how to ensure high availability and scalability when deploying an app 
```
High avalability: 
- load balencer to distribute traffic evenly 
- redundancy - deploy vm's in multiple avalability zones 
- health checks + replacement of unhealthy machines 
- have disaster recovery stratagey in place
- monitoring and alerts 
scalability: 
- set up autoscaling 
``` 
* how to SSH into a VM within a VM Scale Set, especially IP address and port to use 
```
In the Azure Portal, navigate to the VMSS.
Under Instances, click on the individual VM instance you're interested in.
Look for the Public IP Address (if available). If your scale set is configured with a public IP, it will be listed here.
``` 
* how to test auto scaling can recover when a VM is marked as unhealthy (tricky!) 
* what is needed to have ready before creating a VM Scale Set to run the app (no /posts page) 
* 
## quick ways to implement better security for the database VM (i.e. without having to implement 3-subnet architecture) (you could highlight these on your 3-subnet architecture diagram) 
```
removal of public ip 
```

## implementing 3-subnet architecture to provide better security with the database VM 
* familiar with the subnets needed and the purpose of each 
* understand the route table's next hop 
```
next hop is the IP address of the next router (or gateway) the packet must travel through on its way to the destination.
``` 
* what needs to be done for an the NVA to work 
```
A network virtual appliance (NVA) virtual machine (VM) is a virtual machine that supports networking functions in a cloud environment. NVAs are used to control traffic flow between network segments with different security levels
``` 
* implications of marking a subnet as 'private' when you create the subnet on Azure (not just naming it 'private-subnet') 
```
means it has no public ip address, so can't be ssh'd into 
```
* what you need to be careful of when setting up your iptables rules 
```
- must block / reject all except specifically allowed traffic 
- rules must be ordered logically - from most specific to least, so all are evaluated 
- use connection trafficing so you dont block packets related to already established connections `-m state` or `m conntrack`
- locking yourself out of the vm by blocking port 22, either explicitly or with default rules on remote servers 
and more!
``` 
* how to SSH into your database VM 
```
Set up a VM (typically with a public IP) that acts as a jump box or bastion host. You SSH into this jump box first, and then from there, SSH into any VM in the VMSS using its private IP address.
``` 

## SSH 
* ways to securely get a private key onto a VM 
```
copy file path of key from local machine 
add that to the connect via ssh
``` 