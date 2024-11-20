# A Secure Logging And Monitoring System
This project focuses on establishing a secure, centralized logging and monitoring system using the ELK Stack (Elasticsearch, Logstash, Kibana). It involves integrating security event logs, configuring alerts to detect potential security incidents, and developing a comprehensive dashboard to continuously monitor and assess the overall security posture.

### Overview of the Workflow:
1. **Set Up a Logging System Using ELK**:
   - Install Elasticsearch, Logstash, and Kibana.
   - Configure the system to ingest logs from multiple sources (e.g., system logs, web servers, firewalls).
2. **Integrate Security Events and Monitoring**:
   - Gather and process security-related logs such as authentication logs, firewall logs, and intrusion detection alerts.
3. **Create Alerts for Specific Security Incidents**:
   - Configure rules for detecting unauthorized access attempts, port scans, or abnormal user behavior.
4. **Build a Dashboard to Monitor the Security Posture**:
   - Use Kibana to create visualizations and dashboards.

### Step 1: Set Up a Logging System Using ELK

#### Step 1.1: Install Elasticsearch
Elasticsearch stores and indexes log data.

1. Add the Elasticsearch repository and install Elasticsearch:
   ```bash
   sudo apt update
   sudo apt install apt-transport-https openjdk-11-jdk -y
   wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
   echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
   sudo apt update
   sudo apt install elasticsearch -y
   ```

2. Configure Elasticsearch:
   ```bash
   sudo nano /etc/elasticsearch/elasticsearch.yml
   ```

   Modify these parameters:
   ```yaml
   network.host: 0.0.0.0
   discovery.type: single-node
   ```

3. Start and enable Elasticsearch:
   ```bash
   sudo systemctl start elasticsearch
   sudo systemctl enable elasticsearch
   ```

4. Verify installation:
   ```bash
   curl -X GET "localhost:9200/"
   ```

---

#### Step 1.2: Install Logstash
Logstash processes and forwards logs to Elasticsearch.

1. Install Logstash:
   ```bash
   sudo apt install logstash -y
   ```

2. Configure Logstash to collect system logs:
   Create a Logstash configuration file:
   ```bash
   sudo nano /etc/logstash/conf.d/system-logs.conf
   ```

   Add the following:
   ```yaml
   input {
     file {
       path => "/var/log/auth.log"
       start_position => "beginning"
       sincedb_path => "/dev/null"
     }
   }

   filter {
     grok {
       match => { "message" => "%{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:hostname} %{DATA:service}: %{GREEDYDATA:message}" }
     }
   }

   output {
     elasticsearch {
       hosts => ["localhost:9200"]
       index => "system-logs"
     }
     stdout { codec => rubydebug }
   }
   ```

3. Start Logstash:
   ```bash
   sudo systemctl start logstash
   sudo systemctl enable logstash
   ```

4. Verify logs are ingested:
   Check the Elasticsearch index:
   ```bash
   curl -X GET "localhost:9200/system-logs/_search?pretty"
   ```

---

#### Step 1.3: Install Kibana
Kibana visualizes log data stored in Elasticsearch.

1. Install Kibana:
   ```bash
   sudo apt install kibana -y
   ```

2. Configure Kibana:
   ```bash
   sudo nano /etc/kibana/kibana.yml
   ```

   Modify these parameters:
   ```yaml
   server.host: "0.0.0.0"
   elasticsearch.hosts: ["http://localhost:9200"]
   ```

3. Start and enable Kibana:
   ```bash
   sudo systemctl start kibana
   sudo systemctl enable kibana
   ```

4. Access Kibana:
   Open a browser and navigate to `http://<server-ip>:5601`.

---

### Step 2: Integrate Security Events and Monitoring

1. **Collect Authentication Logs**:
   Configure Logstash to collect `/var/log/auth.log` to detect login attempts and sudo actions.

2. **Integrate Firewall Logs**:
   If using `ufw` or `iptables`, redirect their logs to a separate file (e.g., `/var/log/firewall.log`) and configure Logstash to ingest them.

   Example Logstash input for firewall logs:
   ```yaml
   input {
     file {
       path => "/var/log/firewall.log"
       start_position => "beginning"
       sincedb_path => "/dev/null"
     }
   }
   ```

3. **Integrate Intrusion Detection Logs**:
   Install tools like **Fail2Ban** or **OSSEC** and configure Logstash to process their logs.

---

### Step 3: Create Alerts for Specific Security Incidents

1. **Enable Elasticsearch Alerting**:
   Install and enable the **Watcher** feature in Elasticsearch.

2. **Create a Security Alert**:
   Example: Alert for repeated failed login attempts.
   ```bash
   curl -X PUT "localhost:9200/_watcher/watch/failed_login_watch" -H 'Content-Type: application/json' -d'
   {
     "trigger": {
       "schedule": { "interval": "5m" }
     },
     "input": {
       "search": {
         "request": {
           "indices": ["system-logs"],
           "body": {
             "query": {
               "bool": {
                 "must": [
                   { "match": { "message": "Failed password" } }
                 ]
               }
             }
           }
         }
       }
     },
     "condition": {
       "compare": {
         "ctx.payload.hits.total": { "gt": 5 }
       }
     },
     "actions": {
       "log": {
         "logging": {
           "text": "More than 5 failed login attempts detected."
         }
       }
     }
   }'
   ```

3. **Configure Kibana Alerts**:
   Use the **Alerting** feature in Kibana to set up email or Slack notifications for critical incidents.

---

### Step 4: Build a Dashboard to Monitor the Security Posture

1. **Create Visualizations in Kibana**:
   - **Failed Logins**: Create a bar chart showing failed login attempts.
   - **Top Sources of Firewall Denies**: Create a pie chart of source IPs with the most denies.
   - **Active Users**: Create a line chart of successful login trends.

2. **Combine Visualizations into a Dashboard**:
   - Navigate to **Dashboard** in Kibana.
   - Add the visualizations created earlier.
   - Save the dashboard as **Security Monitoring Dashboard**.

3. **Share the Dashboard**:
   - Export the dashboard as a **link** or **PDF**.
   - Grant team members access via Kibana roles.

---

=================

1. **Create Setup Documentation**:
   - Outline the procedures for installing and configuring the ELK Stack.
   - Specify log sources and categories (e.g., authentication, firewall, intrusion detection).
   - Define alerting rules and communication channels for notifications.

2. **Conduct Team Training**:
   - Provide a practical session on using the dashboard and understanding visual data.
   - Offer guidance on modifying Logstash configurations to accommodate new log sources.
   - Provide an overview of Kibana’s Alerting and Watcher features for real-time monitoring.
