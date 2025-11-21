# DevOps Intern Assignment – PowerPlay

## 🟢 Part 1: Environment Setup

### 1️⃣ EC2 Instance Provisioning

- Spin up a **t3.micro Ubuntu Server 22.04 LTS** on AWS EC2.
- Create a new key pair, and configure SSH access.
- We can even use Terraform to provisioning of resources to avoid any personal error.
  
  Used an **SSH Agent** for safe private key handling!

### 2️⃣ User Creation & Sudo Setup

- Create a dedicated Linux user:
  ```bash
  sudo adduser devops_intern
  ```
- Grant **passwordless `sudo` access**:
  ```bash
  sudo visudo
  # Add this line:
  devops_intern ALL=(ALL) NOPASSWD:ALL
  ```

### 3️⃣ Hostname Customization

- Personalize your host for clarity:
  ```bash
  sudo hostnamectl set-hostname yash-devops
  ```

#### 🔍 Deliverables:
- Attached in Part_1 Directory
- `hostname` output, `/etc/passwd` entry for `devops_intern`
- Output of `sudo whoami` as the new user

![My Screenshot](Part_1/Screenshot_2025-11-20_214659.png)
![My Screenshot](Part_1/Screenshot_2025-11-20_215258.png)
![My Screenshot](Part_1/Screenshot_2025-11-20_215326.png)

---

## 🟢 Part 2: Simple Web Service Setup

### 1️⃣ Nginx Installation & Startup

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 2️⃣ Custom HTML Webpage

Place this in `/var/www/html/index.html`:

```html
<html>
<body>
  <h1>Name: Yash Wagh</h1>
  <h2>Instance ID: $(curl -s http://169.254.169.254/latest/meta-data/instance-id)</h2>
  <h3>Uptime: $(uptime -p)</h3>
</body>
</html>
```

> **Restart Nginx** to serve your page:
```bash
sudo systemctl restart nginx
```

#### 🔍 Deliverables:
- Attached in Part_2 Directory
- Screenshot of your webpage – view it at your EC2 public IP!

![My Screenshot](Part_2/Screenshot_2025-11-20_222537.png)
![My Screenshot](Part_2/Screenshot_2025-11-20_222601.png)
![My Screenshot](Part_2/Screenshot_2025-11-20_222622.png)
![My Screenshot](Part_2/Screenshot_2025-11-20_222638.png)
![My Screenshot](Part_2/Screenshot_2025-11-20_222035.png)

---

## 🟢 Part 3: Monitoring Automation

### 1️⃣ System Monitoring Script

Create `/usr/local/bin/system_report.sh`:

```bash
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
```

Set executable permissions:

```bash
sudo chmod +x /usr/local/bin/system_report.sh
```

### 2️⃣ Cron Job Setup

Run every 5 minutes – edit as root:
```bash
sudo crontab -e
# Add this line:
*/5 * * * * /usr/local/bin/system_report.sh >> /var/log/system_report.log 2>&1
```

#### 🔍 Deliverables:
- Attached in Part_3 Directory
- Screenshot of `crontab` configuration
- `system_report.log` with *at least two* entries

![My Screenshot](Part_3/Screenshot_2025-11-20_233605.png)
![My Screenshot](Part_3/Screenshot_2025-11-21_095916.png)
![My Screenshot](Part_3/Screenshot_2025-11-20_233619.png)

---

## 🟢 Part 4: AWS CloudWatch Integration

### 1️⃣ AWS CLI Configuration

Set up CLI and credentials:
```bash
aws configure
```

### 2️⃣ Create Log Group & Stream

```bash
aws logs create-log-group --log-group-name /devops/intern-metrics
aws logs create-log-stream --log-group-name /devops/intern-metrics --log-stream-name intern-stream
```

### 3️⃣ Convert Logs to CloudWatch JSON

```bash
awk 'NF { printf("{\"timestamp\": %d000, \"message\": \"%s\"}\n", systime(), $0); }' \
   /var/log/system_report.log > clean-events.json

echo "[" > clean-array.json
sed '$!s/$/,/' clean-events.json >> clean-array.json
echo "]" >> clean-array.json
```

### 4️⃣ Upload Log Events

```bash
aws logs put-log-events \
    --log-group-name /devops/intern-metrics \
    --log-stream-name intern-stream \
    --log-events file://clean-array.json
```

![My Screenshot](Part_4/Screenshot_2025-11-21_072800.png)
![My Screenshot](Part_4/Screenshot_2025-11-21_072815.png)
![My Screenshot](Part_4/Screenshot_2025-11-21_072716.png)
![My Screenshot](Part_4/Screenshot_2025-11-21_072701.png)

---

— Yash Wagh
