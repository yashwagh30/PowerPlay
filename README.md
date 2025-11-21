Part 1: Environment Setup
1. EC2 Instance

A t3.micro Ubuntu Server 22.04 LTS instance was launched using the AWS EC2 dashboard. A key pair was created for SSH access.

2. User Creation

A new user named devops_intern was created using the following command:

sudo adduser devops_intern

3. Passwordless Sudo Access

The sudoers file was updated:

sudo visudo


The line below was added to allow passwordless sudo:

devops_intern ALL=(ALL) NOPASSWD:ALL

4. Hostname Update

The server hostname was changed to include my name:

sudo hostnamectl set-hostname yash-devops

Deliverables

The screenshots include:

Output of hostname

The devops_intern entry from /etc/passwd

Output of sudo whoami when run as the new user

Part 2: Simple Web Service Setup
1. Installing Nginx

Nginx was installed and enabled:

sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx

2. HTML Page Setup

A simple HTML page was placed at /var/www/html/index.html.
It displays my name, the EC2 instance ID (fetched from metadata), and the server uptime.

<html>
<body>
<h1>Name: Yash Wagh</h1>
<h2>Instance ID: $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</h2>
<h3>Uptime: $(uptime -p)</h3>
</body>
</html>


Nginx was restarted:

sudo systemctl restart nginx

Deliverable

A screenshot of the webpage accessed using the instance's public IP.

Part 3: Monitoring Script
1. System Report Script

A script named system_report.sh was created at /usr/local/bin/system_report.sh:

#!/bin/bash
echo "===== System Report ====="
echo "Date: $(date)"
echo "Uptime: $(uptime -p)"
echo "CPU Usage: $(top -bn1 | grep 'Cpu(s)' | awk '{print $2 + $4}')%"
echo "Memory Usage: $(free | awk '/Mem/ {printf(\"%.2f\", $3/$2 * 100)}')%"
echo "Disk Usage: $(df -h / | awk 'NR==2 {print $5}')"
echo "Top 3 Processes by CPU:"
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -n 4
echo "========================="
echo


The script was made executable:

sudo chmod +x /usr/local/bin/system_report.sh

2. Cron Job

A cron job was created for the root user to run the script every five minutes:

sudo crontab -e


Cron entry:

*/5 * * * * /usr/local/bin/system_report.sh >> /var/log/system_report.log 2>&1

Deliverables

Screenshot of the cron entry

Screenshot showing multiple log entries inside /var/log/system_report.log

Part 4: AWS CloudWatch Integration
1. AWS CLI Setup

AWS CLI v2 was installed manually using the official installer.
The CLI was configured using:

aws configure

2. CloudWatch Log Group and Stream

A log group and log stream were created:

aws logs create-log-group --log-group-name /devops/intern-metrics

aws logs create-log-stream \
--log-group-name /devops/intern-metrics \
--log-stream-name intern-stream

3. Preparing Log Events

System log entries were converted into a CloudWatch-compatible JSON array:

awk 'NF { printf("{\"timestamp\": %d000, \"message\": \"%s\"}\n", systime(), $0); }' /var/log/system_report.log > clean-events.json

echo "[" > clean-array.json
sed '$!s/$/,/' clean-events.json >> clean-array.json
echo "]" >> clean-array.json

4. Upload to CloudWatch

The JSON log events were uploaded using:

aws logs put-log-events \
--log-group-name /devops/intern-metrics \
--log-stream-name intern-stream \
--log-events file://clean-array.json
