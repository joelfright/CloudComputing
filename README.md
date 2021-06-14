# Cloud Computing with AWS

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
