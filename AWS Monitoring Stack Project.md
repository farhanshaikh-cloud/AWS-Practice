Here's a **short AWS Monitoring Project** that is perfect for DevOps practice and can be completed in **1–2 hours**.

# AWS Monitoring Stack Project

## Project Name

**EC2 Monitoring with Prometheus, Grafana, Node Exporter & CloudWatch**

## Architecture

![Image](https://images.openai.com/static-rsc-4/9yTlfWdKjeoz4wxDIuFQIAdlCxvUhikc_z5WVmuwmGgE0IilFj7gxxNbNLKmR4A9TO-LIneXoHYOtHbIsU38TofDxK14i9VEuuOTTSzxi4uaEcP2phkoYybP0EXNWNydkkRmOgiVe-oMPt_WmVliJZpIUxBfVsqw-oW85aqlRfDT3yMpmO0ipWaIpjV5TvHO?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/fFOapMHPH0ZY0jj447uwbdJUQ0xRd06CzBmsRoCnKNm8eWzu2LVHop6brLRxkOuDqanGkNIgcrzVWLiPIYV709OSf8Px9UfbyjpWBTW2XcKPtOYI9OURwXbstqt8utq8Api57JZfmAbypiJo0JY47v7LHJLFzZAsGTuQs32Cuyr5CtGm8Ci7UTy0LgGCFGpv?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/4LTpxk_tWuWNjGvQ601Y0zgAxrawM9Kv4kMKJJjL5d_QYsM2hjHxQL9vG80paF8MIcI2fYtelyP33kKGCJTCJ1ioqOuuNZfnPro77qPCf2_OugzLk6PIvMlRdidt5JUtXZHmmxdZ1HkPqLmAmriycHAHkfXFK1tlVf-Yi4U1cXTMYOJMhvZwa9yekOsW1myk?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/jLsHy5V2LSM799bmDgZlffJuSKGJ3BpFnXn3GImA0_SaXliUXMkn_Z-8i1rdXKflPzOUCYYnwuSl24ymVWChv3Vf-sH8VimjqZw9xM3-qJT6GXhR9MOKBs7MrU08yg_cI4uC7N48bacwSwKOUBX9HJ6kkvFKvdnSmOAhmx3oMO5IsHPygbuICO9qrBh1nIZe?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/9gw1j-3wOvasIuuXTANrnyxaNUKoEQhHUUxQPrpLSuyU4TcSkzMdhshLIPIsEX0qxpOeN2YEK3ewdKrdh6yG2myccw23h6zIhHIcexypZgrfrJ7At9Y07UFE3WptujTO8_5T6XXLvC82HUBcQzVYQWDwyRicKhZzo1Kqp_z-t-JJVyg6fVir27xHPCJt4WqR?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/MNhHXMRWt8ArUv5uSvX1evMEqiJ7TnkUDh7HTDj7hB4QOChhox5S1CnQMWlBycr3z9gBn9KDNaJWCLkpGmMIlvI-9w9t2DriqawGBQfkfnDOqm66utfmkgmvJREVYZTxDtad5JH3EWGlTyYVWVr5Rdel4gdfH-4IC3GCZq_U7tvucYsB8P0zCBgFegwLVYgr?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/kUxkGG8LaQkjpEBC_L5qXP3xvKnQj8qhdc7bZMjxtMjBsRQZ6fTzuOMVQ93HZSpLUaWLTPBlKHuBVDq_qtvrPo6S2t8vWaYnq61c4ueQbIvOj3QBngrS5sSXb3_wMjeTl-4SnODlEy8EfPkTHFiPF_SZ4PYmqWk2Y13RUK_HMrc1QrPlzOyZf3psS-FhUXXl?purpose=fullsize)

```
EC2 Instance
     |
     +--> Node Exporter (System Metrics)
     |
     +--> CloudWatch Agent (AWS Metrics & Logs)
     |
Prometheus ----> Grafana Dashboard
```

---

# Services Used

* Amazon Web Services EC2
* Prometheus
* Grafana
* Node Exporter
* Amazon CloudWatch

---

# Step 1: Launch EC2

Launch an Ubuntu EC2 instance:

* t2.micro (Free Tier)
* Allow ports:

  * 22 (SSH)
  * 3000 (Grafana)
  * 9090 (Prometheus)
  * 9100 (Node Exporter)

---

# Step 2: Install Node Exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz

tar -xvf node_exporter-1.9.1.linux-amd64.tar.gz

cd node_exporter-1.9.1.linux-amd64

./node_exporter &
```

Verify:

```bash
curl localhost:9100/metrics
```

---

# Step 3: Install Prometheus

```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.4.2/prometheus-3.4.2.linux-amd64.tar.gz

tar -xvf prometheus-3.4.2.linux-amd64.tar.gz

cd prometheus-3.4.2.linux-amd64
```

Edit:

```bash
nano prometheus.yml
```

Add:

```yaml
scrape_configs:
  - job_name: node
    static_configs:
      - targets:
          - localhost:9100
```

Start:

```bash
./prometheus --config.file=prometheus.yml
```

Access:

```
http://EC2-PUBLIC-IP:9090
```

---

# Step 4: Install Grafana

```bash
sudo apt update

sudo apt install -y adduser libfontconfig1 musl

wget https://dl.grafana.com/oss/release/grafana_12.0.2_amd64.deb

sudo dpkg -i grafana_12.0.2_amd64.deb

sudo apt --fix-broken install -y

sudo systemctl enable grafana-server

sudo systemctl start grafana-server
```

Access:

```
http://EC2-PUBLIC-IP:3000
```

Default Login:

```
admin
admin
```

---

# Step 5: Add Prometheus Data Source

Grafana → Connections → Data Sources

Add:

```
http://localhost:9090
```

Save & Test

---

# Step 6: Import Dashboard

Dashboard ID:

```text
1860
```

(Node Exporter Full Dashboard)

You can now monitor:

* CPU Usage
* RAM Usage
* Disk Usage
* Network Traffic
* Load Average

---

# Step 7: Install CloudWatch Agent

Configure IAM Role:

```text
CloudWatchAgentServerPolicy
```

Install:

```bash
sudo apt update

wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

sudo dpkg -i amazon-cloudwatch-agent.deb
```

Configure:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

Start:

```bash
sudo systemctl start amazon-cloudwatch-agent
```

---

# Step 8: Verify CloudWatch

Go to:

**CloudWatch → Metrics**

Check:

* CPU Utilization
* Memory Utilization
* Disk Usage
* Network In/Out

---

# Expected Outcome

You will have:

✅ EC2 running on AWS
✅ Node Exporter collecting OS metrics
✅ Prometheus scraping metrics
✅ Grafana visualizing dashboards
✅ CloudWatch collecting AWS metrics and logs

This is a common beginner-to-intermediate DevOps monitoring project and looks good on a resume as **"Implemented centralized monitoring on AWS using Prometheus, Grafana, Node Exporter, and CloudWatch."**
