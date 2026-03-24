
# EC2 Monitoring with CloudWatch

A beginner AWS project that provisions a Linux web server on EC2 and sets up a full monitoring pipeline using CloudWatch — automated CPU alerts, email notifications via SNS, and a live metrics dashboard.

---

## What is happening

1. An **EC2 t3.micro instance** runs Amazon Linux with Apache web server installed.
2. **CloudWatch** automatically collects metrics from the instance every 5 minutes — CPU utilisation, network I/O, and instance health checks.
3. A **CloudWatch Alarm** watches `CPUUtilization`. If the average CPU exceeds **70% for 5 minutes**, it triggers an alert.
4. The alarm publishes a message to an **SNS Topic**, which immediately sends an **email notification** to a subscribed address.
5. A **CloudWatch Dashboard** shows all metrics in one place — CPU, network in/out, and alarm status — updated in real time.

```
EC2 Instance (Apache)
      │
      ▼
CloudWatch Metrics  ──►  CloudWatch Alarm (CPU > 70%)
      │                          │
      ▼                          ▼
  Dashboard                 SNS Topic
  (visual)                      │
                                 ▼
                          Email Notification
```

---

## AWS Services Used

| Service | Purpose |
|---|---|
| **EC2** | Virtual Linux machine running the Apache web server |
| **CloudWatch** | Collects metrics, evaluates alarm conditions, hosts dashboard |
| **SNS** | Delivers email notification when the alarm fires |
| **IAM** | Security group rules controlling inbound traffic (ports 22, 80, 443) |

---
<img width="2343" height="847" alt="image" src="https://github.com/user-attachments/assets/ff9e405c-a5b4-4cee-9d49-77cbf1217fa7" />

## Project Structure

```
ec2-cloudwatch-monitoring/
├── README.md
└── setup-commands.sh       # All terminal commands used on the EC2 instance
```

### setup-commands.sh

```bash
# Update packages
sudo yum update -y

# Install and start Apache
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

# Create a test web page
echo "<h1>Hello from $(hostname)</h1><p>EC2 + CloudWatch demo</p>" \
  | sudo tee /var/www/html/index.html

# Generate CPU load to test the alarm (runs for ~60s)
yes > /dev/null &
yes > /dev/null &
yes > /dev/null &
# Stop load: kill %1 %2 %3
```

---

## Architecture Decisions

- **t3.micro** — Free Tier eligible (750 hours/month). Sufficient for a demo web server.
- **Static threshold alarm** — Simple and readable. Fires when 1 out of 1 data point breaches 70% CPU.
- **SNS email subscription** — Zero-cost alerting with no third-party tools needed.
- **Inline dashboard** — Screenshot-able proof of monitoring for portfolio use.

---

## What Can Be Improved

### Monitoring depth
- **Custom metrics** — CloudWatch does not collect memory or disk usage by default. Install the **CloudWatch Agent** on the EC2 instance to push RAM utilisation and disk I/O as custom metrics.
- **Log ingestion** — Stream Apache access logs (`/var/log/httpd/access_log`) to **CloudWatch Logs** and use **Logs Insights** to query for 404 errors, slow responses, or unusual traffic patterns.

### Alerting
- **Composite alarms** — Combine multiple alarms (high CPU AND low network out) to reduce false positives before paging someone.
- **Multiple notification channels** — Add Slack or PagerDuty as SNS subscribers alongside email for faster incident response.
- **Anomaly detection** — Replace the static 70% threshold with CloudWatch's ML-based anomaly detection band, which adapts to your instance's normal behaviour automatically.

### Infrastructure
- **Auto Scaling** — Attach the EC2 instance to an Auto Scaling Group. CloudWatch alarms can trigger scale-out (add instances) when CPU is high and scale-in (remove instances) when it drops — eliminating manual intervention entirely.
- **Application Load Balancer** — Place an ALB in front of EC2 and monitor `RequestCount`, `TargetResponseTime`, and `HTTP 5xx` metrics from the ALB for application-level observability.
- **Infrastructure as Code** — Re-provision the entire stack using **AWS CloudFormation** or **Terraform** so the setup is reproducible and version-controlled.

### Security
- **Restrict SSH access** — The current security group allows SSH (port 22) from `0.0.0.0/0` (anywhere). In production, restrict this to your specific IP or use **AWS Systems Manager Session Manager** to connect without opening port 22 at all.
- **IAM Instance Profile** — Attach an IAM role to the EC2 instance so it can securely call other AWS services (like writing to S3) without hardcoding credentials on the machine.

---

## Cost

All resources used are within the **AWS Free Tier**:
- EC2 t3.micro — 750 hours/month free for 12 months
- CloudWatch — 10 custom metrics, 10 alarms, 3 dashboards free per month
- SNS — 1,000 email notifications free per month

> **Remember to stop your EC2 instance when not in use** to avoid consuming free tier hours.

---

## Skills Demonstrated

`AWS EC2` `CloudWatch` `SNS` `IAM` `Linux` `Apache` `Infrastructure Monitoring` `Alerting Pipelines`
