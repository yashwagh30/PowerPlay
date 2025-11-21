PART 1 — Environment Setup
1. Launched EC2 Instance

Instance type: t3.micro (free tier)

AMI: Ubuntu Server 22.04 LTS

Region: (your-region)

2. Created a new user
sudo adduser devops_intern

3. Added passwordless sudo

Edited sudoers file:

sudo visudo


Added:

devops_intern ALL=(ALL) NOPASSWD:ALL

4. Changed hostname
sudo hostnamectl set-hostname yash-devops

Deliverables

Screenshot of:

hostname

/etc/passwd showing devops_intern

Output of sudo whoami as devops_intern

PART 2 — Simple Web Service Setup
1. Installed NGINX
sudo apt update
sudo apt install nginx -y

2. Created HTML page

Path: /var/www/html/index.html

<html>
<body>
<h1>Name: Yash Wagh</h1>
<h2>Instance ID: $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</h2>
<h3>Uptime: $(uptime -p)</h3>
</body>
</html>


Restarted NGINX:

sudo systemctl restart nginx

3. Deliverable

Screenshot of webpage from EC2 public IP

PART 3 — Monitoring Script
1. Script

File: /usr/local/bin/system_report.sh

#!/bin/bash
echo "----- System Report ------"
echo "Date: $(date)"
echo "Uptime: $(uptime -p)"
echo "CPU Usage: $(top -bn1 | grep 'Cpu(s)' | awk '{print $2 + $4}')%"
echo "Memory Usage: $(free | awk '/Mem/ {printf(\"%.2f\", $3/$2 * 100)}')%"
echo "Disk Usage: $(df -h / | awk 'NR==2 {print $5}')"
echo "Top 3 Processes by CPU:"
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -n 4
echo "---------------------"
echo


Made executable:

sudo chmod +x /usr/local/bin/system_report.sh

2. Cron Job
sudo crontab -e


Cron entry:

*/5 * * * * /usr/local/bin/system_report.sh >> /var/log/system_report.log 2>&1

3. Deliverables

Cron configuration screenshot

Screenshot of /var/log/system_report.log with entries

PART 4 — AWS CloudWatch Integration
1. Created log group
aws logs create-log-group --log-group-name /devops/intern-metrics

2. Created log stream
aws logs create-log-stream \
--log-group-name /devops/intern-metrics \
--log-stream-name intern-stream

3. Generated clean log events JSON
awk 'NF { printf("{\"timestamp\": %d000, \"message\": \"%s\"}\n", systime(), $0); }' /var/log/system_report.log > clean-events.json


Converted to JSON array:

echo "[" > clean-array.json
sed '$!s/$/,/' clean-events.json >> clean-array.json
echo "]" >> clean-array.json

4. Uploaded logs
aws logs put-log-events \
--log-group-name /devops/intern-metrics \
--log-stream-name intern-stream \
--log-events file://clean-array.json
