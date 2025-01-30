# Azure - now with more azure 

- [Azure - now with more azure](#azure---now-with-more-azure)
  - [Create a new VM with Ubunutu 22.04 LTS:](#create-a-new-vm-with-ubunutu-2204-lts)
    - [basic](#basic)
    - [disk](#disk)
  - [copy sparta test app to VM](#copy-sparta-test-app-to-vm)
    - [Using scp](#using-scp)
  - [add port rule + create new network security group](#add-port-rule--create-new-network-security-group)
    - [using github](#using-github)
  - [Deploy the Sparta test app](#deploy-the-sparta-test-app)
  - [making a generalised image](#making-a-generalised-image)
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
    - [next steps](#next-steps)
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

## add port rule + create new network security group 
![add a port rule](25.01.27/create-nw-security-group.png)

### using github 

*to do*

## Deploy the Sparta test app

* cd into app 
* npm install 
* check permissions over app folder (should have full) 
* node app.js or npm start 
* ctrl c to exit 
* webadress:3000 to view 

## making a generalised image 

* Move your app code from adminuser home directory to root directory (so in the root directory you will have a "repo" folder, containing an "app" folder)
* waagent command that will wipe out the adminuser folder `waagent -deprovision+user` in command line 
* stop and wait for VM to deprovision 
* make an image in azure: capture > image (in menu)
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

bindIP: 0.0.0

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


echo 'export DB_HOST=mongodb://20.90.208.188:27017/posts' >> /.bashrc

### next steps 
   21  export DB_HOST=mongodb://10.0.3.4:27017/posts  >>.bashrc
   22  printenv DB_HOST
   23  npm install 
   24  node app.js
   25  cd /
   26  ls
   27  cd repo
   28  ls
   29  node app.js
   30  cd app
   31  node app.js
   32  printenv DB_HOST
   33  export DB_HOST=mongodb://10.0.3.4:27017/posts
   34  printenv DB_HOST
   35  npm install 
   36  node app.js
   37  history 
 
    1  pwd
    2  cd /
    3  ls
    4  cd repo
    5  ls
    6  cd app 
    7  npm start
    8  sudo apt-get update -y
    9  sudo apt-get upgrade -y 
   10  sudo systemct1 restart nginx
   11  sudo systemctl restart nginx
   12  sudo systemctl status nginx
   13  node app.js
   14  cd /
   15  ls
   16  cd repo
   17  ls
   18  cd app
   19  ls
   20  node app.js
   21  export DB_HOST=mongodb://10.0.3.4:27017/posts
   22  printenv DB_HOST
   23  npm install 
   24  node app.js
   25  cd /
   26  ls
   27  cd repo
   28  ls
   29  node app.js
   30  cd app
   31  node app.js
   32  printenv DB_HOST
   33  export DB_HOST=mongodb://10.0.3.4:27017/posts
   34  printenv DB_HOST
   35  npm install 
   36  node app.js
   37  history 
   38  close
   39  cd /
   40  ls
   41  ls -a
   42  sudo ls -a
   43  cd media
   44  ls
   45  cd ..
   46  cd var
   47  ls
   48  cd ..
   49  ls
   50  cd etc
   51  ls
   52  nginx
   53  cd nginx
   54  ls
   55  sudo nano proxy_params 
   56  cd
   57  ls
   58  cd /
   59  ls
   60  cd repo
   61  ls
   62  cd app 
   63  npm start
   64  cd /ect/nginx
   65  cd /
   66  ls
   67  cd etc/nginx/
   68  ls
   69  nano proxy_params 
   70  sudo nano proxy_params 
   71  cd sites-available/
   72  ls
   73  cd ..
   74  nano nginx.conf 
   75  cd sites-available/
   76  nano default
   77  sudo nano default
   78  cd /repo/app
   79  ls
   80  npm start
   81  sudo nginx -t
   82  sudo systemctl reload nginx
   83  sudo systemctl status nginx
   84  npm start
   85  npm install pm2 -g
   86  sudo npm install pm2 -g
   87  pm2 start /usr/sbin/nginx --name nginx
   88  pm2 list
   89  pm2 restart nginx
   90  pm3 logs nginx
   91  pm2 logs nginx
   92  sudo pm2 start /usr/sbin/nginx --name nginx
   93  pm2 start app.js
   94  sudo systemctl start nginx
   95  history


    1  sudo apt-get update -y
    2  sudo apt-get upgrade -y
    3  history
    4  cd /
    5  pwd
    6  ls
    7  sudo apt-get install gnupg curl 
    8  curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc |    sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg    --dearmor
    9  echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
   10  sudo apt-get update
   11  sudo apt-get install -y mongodb-org=7.0.6 mongodb-org-database=7.0.6 mongodb-org-server=7.0.6 mongodb-mongosh mongodb-org-mongos=7.0.6 mongodb-org-tools=7.0.6
   12  mongodp --version
   13  nano deply_db.sh
   14  ls
   15  cd 
   16  ls
   17  nano deply_db.sh
   18  sudo systemctl status mongod
   19  sudo systemctl start mongod
   20  sudo systemctl status mongod
   21  sudo nano /etc/mongod.conf 
   22  sudo systemctl restart mongod
   23  exit
   24  sudo systenctl is-enabled mongo db
   25  sudo systemctl is-enabled mongo db
   26  sudo systemctl is-enabled mongod
   27  sudo systemctl start mongod
   28  sudo systemctl is-enabled mongod
   29  sudo systemctl enabled mongod
   30  sudo systemctl enable mongod
   31  sudo systemctl start mongod
   32  sudo systenctl status mongo db
   33  sudo systemctl status mongo db
   34  sudo systemctl status mongod
   35  close
   36  ls
   37  nano deply_db.sh 
   38  cd /
   39  ls
   40  status nginx
   41  sudo systemctl status mongod
   42  cd 
   43  nano deply_db.sh 
   44  sudo systemctl start mongod
   45  printenv DB_HOST
   46  export DB_HOST=mongodb://10.0.3.4:27017/posts  >>.bashrc
   47  printenv DB_HOST
   48  cd
   49  ls
   50  cd /
   51  ls
   52  cd etc
   53  ls
   54  sudo systemctl start mongod
   55  ls
   56  nano deply_db.sh 
   57  sudo nano /etc/mongod.conf
   58  sudo systemctl status mongod
   59  sudo apt-get update -y
   60  sudo apt-get upgrade -y 
   61  sudo systemctl restart mongod
   62  sudo systemctl status mongod
   63  sudo apt-get update
   64  printenv DB_HOST
   65  history
   66  export DB_HOST=mongodb://10.0.3.4:27017/posts  >>.bashrc
   67  printenv DB_HOST
   68  sudo nano .bashrc
   69  echo 'export DB_HOST=mongodb://10.0.3.4:27017/posts' >> ~/.bashrc
   70  source ~/.bashrc
   71  sudo nano .bashrc
   72  history


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