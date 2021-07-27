# AWS 2-Tier Architecture Node App Deployment with Mongodb Database

-  A 2-Tier architecture environment will be created to carry out all the functionalities of an 
application.
- This includes an EC2 instance for the database mongodb, and the EC2 instance to run the app.
![alt text](2Tier_app_deployment_on_aws_diagram.png)


## EC2 Instance (Node Application)
- Firstly we set up an ec2 instance with the following information.
##### AMI
`Ubuntu Server 16.04 LTS (HVM), SSD Volume Type`
##### Instance Type
`t2.micro`
##### Configure Instance 
```
Subnet:   subnet ............default1A
Auto-assign Public IP: Enable
```
##### Add Tags
``` 
Name: eng89_shervin
Description: Machine for running Application  
```
##### Configure Security Group 
- First Inbound protocol should be for the `SSH` at default port 22 and the localhost `IPV4`.
- `Custom TCP` at port 3000 to allow access for node app.
- ` HTTP` protocol also for global access.

### Running EC2 Instance and Installing Dependencies

- To get into instance on localhost, locate the folder were the ssh key is present.
- Sync the folder were application is present using the following command `scp -i pem_file_path -r app_file_path machine_name: file_name_in_instance`. For example `scp -i ~/.ssh/eng89_devops.pem -r app/ ubuntu@63.32.94.140:~/app/`
- Run the `chmod` command that can be found on aws by selecting the instance and pressing connect. The command is present under SSH Client.
- SSH into machine and download the following dependencies one by one.

```python
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install nginx -y
sudo apt-get install git -y
sudo apt-get install python-software-properties -y
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install nodejs -y
sudo npm install pm2 -g
```
- The app can now be run by entering the `App` folder and typing
```python
npm install
node app.js
```
- The app can now be seen on your AWS public IP Address with the extension :3000.

#### Final Step (setting up a reverse proxy)
- A reverse Proxy can easily be setup by entering and configuring the default file in nginx.
```python
cd etc/nginx/sites-available
rm -rf default
sudo nano default
```
- The default file should have the following present.
```python
server{
       listen 80;
       server_name _;
       location / {
       proxy_pass http://34.240.32.57:3000/;   # This proxy Must be your Instance Public IP Address
       proxy_http_version 1.1;
       proxy_set_header Upgrade \$http_upgrade;
       proxy_set_header Connection 'upgrade';
       proxy_set_header Host \$host;
       proxy_cache_bypass \$http_upgrade;
}}
```
- You should now be able to see the app by restarting nginx via `sudo systemctl restart nginx` and running the app, without the extension :3000.


## EC2 Instance (Mongodb Database)
- Firstly we set up an ec2 instance with the following information.
##### AMI
`Ubuntu Server 16.04 LTS (HVM), SSD Volume Type`
##### Instance Type
`t2.micro`
##### Configure Instance 
```
Subnet:   subnet ............default1A
Auto-assign Public IP: Enable
```
##### Add Tags
``` 
Name: eng89_shervin_db
Description: Databse for node application.  
```
##### Configure Security Group 
- First Inbound protocol should be for the `SSH` at default port 22 and the localhost `IPV4`.
- `Custom TCP` at port 27017 to at node app instance IP Address for access to and from first EC2 instance.
- ` HTTP` protocol also for global access.

### Running EC2 Instance and Installing Dependencies
- To run and enter is the same as the first instance.
- The dependencies to be installed are the following.
```python
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install mongodb-org=3.2.20 -y
sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20
```
- We must now configure MongodB and restart the system. We do this via the following.
```python
cd /etc
sudo nano mongod.conf
```
- Change the BindIp to the following `bindIp: 0.0.0.0`
- We now restart Mongodb via `sudo systemctl restart mongodb`.



## Running Node App and MongoDb
Everything is set up. The Last steps are the following.
- Enter app node EC2 Instance an create env variable via `export DB_HOST' => 'mongodb://MongoDB_IP_ADDRESS:27017/posts`
- Enter app folder and seed data via `node seeds/seed.js`
- The app can now be seen running on the node app IP address with extension /Posts.
