# Cloud Computing with AWS

### Cloud Computing and AWS

#### What is cloud computing, what are the benefits and who is using it?

Cloud computing is the on-demand delivery of IT resources over the Internet with pay-as-you-go pricing. Instead of buying, owning, and maintaining physical data centers and servers, you can access technology services, such as computing power, storage, and databases, on an as-needed basis from a cloud provider. The benefits are flexibility, efficiency and strategic value. Many people use it in their day to day life, from gaming on steam to streaming on netflix.

#### What is AWS, what are the advantages and who uses it?

Amazon Web Services is a subsidiary of Amazon providing on-demand cloud computing platforms and APIs to individuals, companies, and governments, on a metered pay-as-you-go basis. AWS is seperating into 3 sections, infrastucture as a service (IaaS), platform as a service (PaaS) and software as a service (SaaS). The advantages are that its easy to use, flexible and secure. It is used by many large companies such as netflix, amazon and ebay.

### Setting up an EC2 instance and begin the node application

1) First order of business is setting up the EC2 instance, you do this by accessing the AWS website and going through all the steps required to begin the instance, including setting ports for HTTP 80, SSH 22 and port 3000.
2) After this you can start the instance, you then want to copy the `app` folder to the virtual machine, to do this you use `scp -i devop_bootcamp.pem -r app/ ubuntu@[iPv4 DNS]:~/.`. This will copy the files over, make sure to be in your ssh folder before you run this.
3) You will also want a `provision.sh` file within the folder to ensure you can run the installation for nodejs. This may look something like this:
```
#!/bin/bash

sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install nginx -y
sudo service nginx restart
sudo systemctl restart nginx
sudo apt-get install nodejs -y
sudo apt-get install python-software-properties -y
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install nodejs -y
sudo npm install pm2 -g
cd app
npm install
```
4) In order to ssh into the file use `ssh -i devop_bootcamp.pem ubuntu@[iPv4 DNS]` you can then affect the virtual machine from there.
4) Next you can start the application by visiting `cd app` and typing `node app.js`. The file should now be on the port 3000 of your public ip.
5) If you wish to reverse proxy you must change your default file in `/etc/nginx/sites-available/default` and replace with this:
```
server {
        listen 80;
        server_name _;
        location / {
                proxy_pass http://(public ip):3000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }
}
```

### Setting up a AWS with a MongoDB database

1) In order to setup with a mongodb database you must first complete all prior steps. Then you must go into the EC2 management console and begin a new instance, with this you must set it up and use the inbound rules as your SSH 22 and the db connection ip which is your app ip which has the port 27017.
2) You must then install a provision file onto your database ssh by using this provision file. Don't forget, if you have ported this via the scp command you must use `sed -i -e 's/\r$//' scriptname.sh` in order to fix the dos to linux error.
```
#!/bin/bash

wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install -y mongodb-org
sudo systemctl restart mongod
sudo systemctl enable mongod
```
3) You must then ssh into your database and change the mongod.conf by going to `/etc/mongod.conf` and changing it to what is seen below, this will allow for the mongo database to connect to the app.
```
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true

systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

net:
  port: 27017
  bindIp: 0.0.0.0
```
4) You must then enter you app via ssh and go into `/etc/environment` where you must input the line `export DB_HOST=mongodb://(your database ip):27017/posts`. This will then persist the environment variable.
5) Then finally you can seed the database by entering `app/seeds` and running `node seeds.js`, then following steps prior enter `app/` and run `node app.js`. The /posts webpage should now appear in your browser.
