Step 1: Install Proxmox Exporter

Prometheus requires an exporter to gather metrics from Proxmox.

Install the Proxmox Exporter:
      SSH into your Proxmox server.
      Run the following command to install the Proxmox exporter:

     apt update/
     apt install python3-pip/
     pip3 install proxmoxer/
     pip3 install prometheus-client/

Clone the Proxmox exporter repository:

git clone https://github.com/znerol/prometheus-proxmox-exporter.git

     cd prometheus-proxmox-exporter\
     python3 setup.py install\

Configure the Proxmox exporter by creating a configuration file (/etc/proxmox-exporter/config.yaml):

    default:
      api_host: "https://<your-proxmox-ip>:8006"
      api_user: "root@pam"
      api_password: "<your-password>"

Replace <your-proxmox-ip> and <your-password> with your Proxmox server details.

Start the Exporter:

Run the exporter as a background service:

        nohup prometheus-proxmox-exporter &

Confirm it’s running by visiting http://<your-proxmox-ip>:9200/metrics.

Step 2: Set Up Prometheus

Prometheus will scrape metrics from your Proxmox exporter.

Install Prometheus:
        On a monitoring server (can be the same Proxmox host), run:

    apt update
    apt install prometheus

Configure Prometheus:

Edit the configuration file /etc/prometheus/prometheus.yml and add your Proxmox exporter:

    scrape_configs:
      - job_name: 'proxmox'
        static_configs:
          - targets: ['<your-proxmox-ip>:9200']

Restart Prometheus:

    systemctl restart prometheus

Test Prometheus:
        Open the Prometheus web interface at http://<prometheus-server-ip>:9090.

Step 3: Visualize Metrics with Grafana

Grafana will display your Proxmox metrics in a user-friendly dashboard.

Install Grafana:
        Follow the official Grafana installation guide.

Connect Grafana to Prometheus:
        Open Grafana at http://<grafana-ip>:3000.\
        Go to Configuration > Data Sources > Add data source.\
        Choose Prometheus and enter the URL: http://<prometheus-server-ip>:9090.

Import a Proxmox Dashboard:
        Visit Grafana Dashboards.\
        Search for "Proxmox" dashboards.\
        Copy the dashboard ID and import it in Grafana (Dashboard > Import).

Step 4: Automate with GitHub

To keep configurations and updates managed, you can use GitHub for your monitoring setup.

Create a Repository:
        Create a private/public repository on GitHub for your Proxmox monitoring configurations.

Push Configuration Files:
        Add the Prometheus and Grafana configuration files to the repository:

    git init
    git remote add origin https://github.com/<your-username>/<repo-name>.git
    git add .
    git commit -m "Add Proxmox monitoring setup"
    git push -u origin main

Set Up GitHub Actions for CI/CD:

Add a GitHub Actions workflow to deploy configurations or updates automatically:
        Create a .github/workflows/deploy.yml file in your repository:

            name: Deploy Monitoring Config

            on:
              push:
                branches:
                  - main

            jobs:
              deploy:
                runs-on: ubuntu-latest
                steps:
                  - name: Checkout repository
                    uses: actions/checkout@v3

                  - name: Deploy Prometheus Config
                    run: |
                      scp prometheus.yml user@<proxmox-ip>:/etc/prometheus/prometheus.yml
                      ssh user@<proxmox-ip> "systemctl restart prometheus"

 Test the Workflow:
        Push changes to your GitHub repository and ensure the workflow successfully deploys your updates.

Step 5: (Optional) Add Alerts

Set up Prometheus alerting rules and Grafana notifications for critical metrics:

Edit the Prometheus alerting rules (/etc/prometheus/alerts.yml) for CPU, RAM, or disk usage:

    groups:
      - name: proxmox-alerts
        rules:
          - alert: HighCPUUsage
            expr: node_cpu_seconds_total{job="proxmox"} > 80
            for: 2m
            labels:
              severity: warning
            annotations:
              summary: "High CPU usage on Proxmox node"

Configure Grafana to send notifications via email, Slack, or Telegram.

With this setup, your Proxmox server metrics will be monitored, visualized, and version-controlled with GitHub!
