# Azure - now with more azure 

- [Azure - now with more azure](#azure---now-with-more-azure)
  - [Create a new VM with Ubunutu 22.04 LTS:](#create-a-new-vm-with-ubunutu-2204-lts)
    - [basic](#basic)
    - [disk](#disk)
  - [copy sparta test app to VM](#copy-sparta-test-app-to-vm)
    - [Using scp](#using-scp)
    - [using github](#using-github)
  - [Deploy the Sparta test app](#deploy-the-sparta-test-app)
  - [Making a generalised image](#making-a-generalised-image)
    - [In terminal](#in-terminal)
    - [On azure](#on-azure)
- [28/01/25](#280125)
  - [making a database VM](#making-a-database-vm)
    - [dependancys:](#dependancys)
    - [bash script:](#bash-script)
      - [script for db:](#script-for-db)
      - [script for app:](#script-for-app)
        - [change bind db](#change-bind-db)
          - [get reverse proxy working - nginx rerout from port 80 to port 3000 (so no longer need to do :3000):](#get-reverse-proxy-working---nginx-rerout-from-port-80-to-port-3000-so-no-longer-need-to-do-3000)
          - [run app in background:](#run-app-in-background)
          - [create db image:](#create-db-image)
          - [user data script](#user-data-script)
    - [farah's sudo debugging](#farahs-sudo-debugging)
  
add: 
* export command persistant vs non 
* what is in dp or app vm (make clear)
## Create a new VM with Ubunutu 22.04 LTS: 

to do: should have a bash script when opening a new VM to update and upgrade <br>
![bash script](25.01.27/make-bash-script.png)
### basic 
![basic settings 1](25.01.27/instance-details-setup.png)
![basic settings 2](25.01.27/admin-account-setup.png)
### disk 
![disk setup screen](25.01.27/disk-vm-setup.png)
## copy sparta test app to VM 
![local-machine to virtual-machine documentation](25.01.27/local-machine-to-VM.png)
### Using scp 
`scp -r nodejs20-sparta-test-app azureuser@20.77.113.224:home/azureuser` 
<br>
-r flag needs to be used to copy directory contents. <br>
azure@user specifies user you want to log in as to the VM <br>
<br>
**or using ssh**: <br>
`scp -i ~/.ssh/tech501-emily-az-key -r nodejs20-sparta-test-app azureuser@20.77.113.224:home/azureuser`
<br>
-i (identity flag), in this case used to add a ssh 

### using github 
1. `git init` the local folder holding the app code 
2. make a corrisponding github repo and push the code to it
3. Clone the repository and navigate into it:
    
    `git clone https://github.com/<your-repo>.git`
    
    `cd <repo-name>`
4. now you should be able to run the app

## Deploy the Sparta test app

1. cd into app `cd /repo/app`
2. run `npm install`
3. check permissions over app folder `ls -l /repo/app` you should have full permissions
4. run the app using either `node app.js` or `npm start` 
5. webadress:3000 to view in browser

## Making a generalised image 

### In terminal 
* Move your app code from adminuser home directory to root directory (so in the root directory you will have a "repo" folder, containing an "app" folder)
* waagent command that will wipe out the adminuser folder `waagent -deprovision+user` in command line
### On azure 
* stop the vm and wait for it to dealocate
  * will also happen if you just click 'capture > image'
* make an image in azure: capture > image
![make an image](25.01.27/capture-vm-image.png)
* setting up the image: 
  *  No need to have the image in the gallery 
  *  name image: tech501-emily-sparta-app-ready-to-run

# 28/01/25 

## making a database VM 
1. set up a new VM: 
### dependancys:

```
    Dependencies
        Name: tech501-yourname-sparta-app-db-vm
        Ubuntu 22.04 LTS image
        Same size as usual
        NSG: allow SSH
        Public IP: yes
        VNet: 2-net one, subnet: private-subnet
    Login & run update & upgrade
```

### bash script: 
#### script for db:
https://www.mongodb.com/docs/v7.0/tutorial/install-mongodb-on-ubuntu/#std-label-install-mdb-community-ubuntu


```
#!bin/bash
#update packages
sudo apt-get update -y
#upgrade packages
sudo apt-get upgrade -y 
#install mongo db 
sudo apt-get install gnupg curl 
# import gpg key 
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
#create file list 
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
# reload package db 
sudo apt-get update
# Install MongoDB Community Server
sudo apt-get install -y mongodb-org=7.0.6 mongodb-org-database=7.0.6 mongodb-org-server=7.0.6 mongodb-mongosh mongodb-org-mongos=7.0.6 mongodb-org-tools=7.0.6
#start
sudo systemctl start mongod
#configure the bind ip 
#work out script from the change bind id bellow 
#restart the mongo db service
sudo systemctl restart mongod 
```
#### script for app:
```
sudo apt install nginx 
> >sudo systemctl restart nginx <br>
> >status nginx <br>
> >sudo systemctl enable nginx <br>
```
cd into app file, npm install 
then start in background: pm2 start app.js

##### change bind db 

`sudo nano /etc/mongod.conf`

bindIP: 0.0.0.0 

*add screenshot of file 
- app connects to db (yay)

###### get reverse proxy working - nginx rerout from port 80 to port 3000 (so no longer need to do :3000): 

should be: 
Backup the nginx configuration file at /etc/nginx/sites-available/default
In the original config file, edit the default location to the following: 

```
#to get into nginx config file

sudo nano /etc/nginx/sites-available/default

#remove try_files line
#replace with:
    proxy_pass http://localhost:3000;}

```
* can also put the actual host ip, but not ideal for image creation

```
#check nginx config file syntax is okay

sudo nginx -t

#reloads nginx to put new edit into place

sudo systemctl reload nginx
```
* check it now works! app should now be default page. 


###### run app in background: 
```
Work out ways to both run, stop and re-start the app in the background (besides using the "&" at the end of the command):

    One way should use pm2
    If time: One other way (can you find another package manager do it like pm2?)
    You should have already used "&" at the end the command to run the app in the background (and you should have already documented the issue with using this method when it comes to stopping/re-starting the app)
    Document the methods you got working

Check the app is working in your browser at the IP address of the VM with :3000 appended to the end (or without port 3000 in the URL if your reverse proxy is running).

Deliverable: Paste link to working app in the main chat with a message like "app running with pm2"
```

```
        sudo npm install -g pm2
        pm2 --version
        [navigate to app]
        pm2 start app.js
        pm2 stop app.js
```
* -g global flag, enables pm2 to be used by all users, not just you. 

###### create db image: 
```
Checklist/plan for creating an app + DB images

☝️ Note: You may have already done some of the first steps. If so, work out where you are up to, then continue.

1. Step 1: Create DB VM using custom base image and manually provision the database
    > log into db vm and run:
    #update packages
    sudo apt-get update -y
    #upgrade packages
    sudo apt-get upgrade -y 
    #restart mongodb
    sudo systemctl restart mongod

2. Step 2: Test database setup correctly
    What things need to be checked?
    check: 
    sudo systemctl status mongod
3. Step 3: Create app VM using custom base image and manually provision the app: 
    * make in azure, then run: 
    #update packages
    sudo apt-get update -y
    #upgrade packages
    sudo apt-get upgrade -y
    **make sure the app runs without the database connection:**
    #restart nginx 
    sudo systemctl restart nginx
    #start the app in background 
    pm2 start app.js 
    make sure the app runs with the database connection: (make sure the DB_HOST variable has the correct IP)
    set DB_HOST variable - to private ip for db 
    cd into app folder
    npm install 
    sudo systemctl restart nginx
4. Step 4: Test
        check public IP to bring up app homepage
        check /posts page
5. Step 5: Create DB VM image from DB VM
        create image 
        delete the original DB VM
6. Step 6: Create DB VM from the DB image just created
7. Step 7: Create app VM image from app VM
        delete the app VM
    Step 8: Create app VM from the app image just created
        use short Bash script in user data (DO NOT WORRY ABOUT BIT - this is for later), instead of this section, login to your app VM to test your app runs (including the post page)
            named run-app-only.sh in your documentation repo
            starts with she-bang
            includes export DB_HOST
            cd into app folder
            (probably don't need it unless you want to seed the database or check your db connection) npm install
            pm2 stop all
            pm start app.js
            check /posts page works (connecting to database VM made from image)


To deliver

    As soon as you have it working...
        Delete the both VMs used to make the images (Leave your app and DB images!)
        Finish documentation
        Submit link to all your documentation on how to automate the deployment, which should be a culmination of all the ways and things you've learnt from how to do a 2-tier deployment of the Sparta test app
``` 
###### user data script

```
#!/bin/bash
 
# navigating into app folder
cd /repo/nodejs20-sparta-test-app/app
 
#export DB_HOST= correct private IP
export DB_HOST=mongodb://*10.0.3.4*:27017/posts
 
#starting the app
pm2 start app.js
 
(check your IP address though)
 ``` 

echo 'export DB_HOST=mongodb://20.90.208.188:27017/posts' >> /.bashrc


### farah's sudo debugging 

    1  cd /
    2  ls
    3  cd repo/nodejs20-sparta-test-app/app/
    4  pm2 start
    5  pm2 start app.js
    6  pm2 stop app.js
    7  export DB_HOST=mongodb://10.0.3.4:27017/posts
    8  printenv DB_HOST
    9  pm2 start app.js
   10  exit
   11  cd /
   12  cd repo/nodejs20-sparta-test-app/app/
   13  pm2 restart app.js --update-env
   14  cd ~
   15  ls
   16  cd /
   17  ls
   18  cd /etc/
   19  ls
   20  cd nginx/
   21  ls
   22  cat nginx.conf
   23  ls
   24  cd /etc/
   25  ls
   26  cd /
   27  cd repo/nodejs20-sparta-test-app/app/
   28  pm2 status
   29  printenv DB_HOST
   30  export DB_HOST=mongodb://10.0.3.4:27017/posts
   31  printenv DB_HOST
   32  pm2 stop app.js
   33  printenv DB_HOST
   34  pm2 start app.js
   35  pm2 stop app.js
   36  npm install
   37  ls
   38  ls -la
   39  sudo
   40  sudo npm install
   41  sudo -E npm install
   42  sudo -E pm2 start app.js
   43  sudo pm2 stop all
   44  sudo -E pm2 start app.js
   45  pm2 stop all
   46  sudo pm2 stop all
   47  sudo -E pm2 start app.js
   48  sudo pm2 stop all
   49  sudo -E npm install
   50  sudo -E pm2 start app.js
   51  sudo pm2 stop all
   52  sudo -E pm2 start app.js
   53  ps aux
   54  sudo kill 1508
   55  ps aux
   56  sudo kill 1558
   57  ps aux
   58  sudo kill 1051
   59  sudo -E npm install
   60  sudo -E pm2 start app.js
   61  history