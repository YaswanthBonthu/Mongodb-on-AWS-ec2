MongoDB Deployment and Automation on AWS:
Deployment Steps (README.md)
1. Provision an EC2 Instance
Launch EC2 Instance:

Launch an Ubuntu or Amazon Linux 2 EC2 instance in your preferred region.
Choose the desired instance type (e.g., t2.micro for testing).
Select a security group that allows inbound traffic on ports:
SSH (port 22) for access to the EC2 instance.
MongoDB (port 27017) to access MongoDB remotely.
Access the EC2 Instance:

Connect to the EC2 instance using SSH:
ssh -i your-key.pem ubuntu@your-ec2-public-ip

2. Install and Configure MongoDB
Install MongoDB Community Edition:

Import the MongoDB public GPG key and add the repository:

curl -fsSL https://pgp.mongodb.org/server-4.4.asc | sudo tee /usr/share/keyrings/mongodb-server-key.asc > /dev/null
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-key.asc] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
sudo apt update
sudo apt install -y mongodb-org
Start the MongoDB Service:

Start and enable MongoDB to run at startup:
sudo systemctl start mongod
sudo systemctl enable mongod
Allow External Access:

Modify /etc/mongod.conf to allow external access:

sudo nano /etc/mongod.conf
Find the bindIp configuration and change it from 127.0.0.1 to 0.0.0.0 for all IPs:

net:
  bindIp: 0.0.0.0
Restart MongoDB:

sudo systemctl restart mongod
3. Automate MongoDB Backup
Create Backup Script (mongodump.sh):

Write a simple Bash script (mongo_backup.sh) to back up MongoDB data:

#!/bin/bash
TIMESTAMP=$(date +\%F_\%T)
BACKUP_DIR="/home/ubuntu/mongodb-backups"
mkdir -p $BACKUP_DIR
mongodump --host 127.0.0.1 --port 27017 --out $BACKUP_DIR/mongo-backup-$TIMESTAMP
Schedule the Backup with Cron:

Edit the crontab to run the backup script every 30 minutes:

crontab -e
Add the following line:

*/30 * * * * /home/ubuntu/mongo_backup.sh
4. Enable Basic Monitoring with CloudWatch
Install CloudWatch Agent:

Download and install the CloudWatch Agent on the EC2 instance:

sudo wget https://d1vvhvl2y92vvt.cloudfront.net/linux/amazon-cloudwatch-agent.deb
sudo dpkg -i amazon-cloudwatch-agent.deb
Configure CloudWatch Agent:

Create and configure the CloudWatch agent by downloading and modifying the configuration file:


sudo wget https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-agent-samples/master/linux/amazon-cloudwatch-agent.json -O /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json
Start the CloudWatch agent:

sudo amazon-cloudwatch-agent-ctl -a start -c file:/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json -s
Verify Monitoring:

Check the CloudWatch console for the EC2 instance metrics, including CPU and memory usage.
