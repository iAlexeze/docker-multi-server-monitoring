# Multi-Server Monitoring with Prometheus, Grafana, Loki, cAdvisor, and Node Exporter

This project sets up a monitoring and logging stack using **Prometheus**, **Grafana**, **Loki**, **cAdvisor**, and **Node Exporter** to collect, visualize, and log metrics from multiple servers and containers. The stack is deployed using **Docker Compose** and includes configurations for monitoring Docker containers, system-level metrics, and logs from multiple remote servers.

## Project Structure

```
.
├── LICENSE
├── README.md
├── docker-compose-agents.yml       # Docker Compose configuration for agent servers
├── docker-compose-control.yml         # Docker Compose configuration for the control server (STG01)
├── logging
│   ├── loki
│   │   └── loki-config.yaml         # Configuration file for Loki
│   └── promtail
│       ├── agents-server
│       │   └── promtail-config.yml  # Promtail configuration for agent servers
│       └── control-server
│           └── promtail-config.yml  # Promtail configuration for the control server
├── monitoring
│   ├── grafana
│   │   ├── grafana.env              # Environment variables for Grafana
│   │   └── provisioning
│   │       ├── dashboards
│   │       │   ├── dashboard.yml    # Grafana dashboard configuration
│   │       │   ├── docker-host-monitoring.json  # Docker host monitoring dashboard
│   │       │   └── node-exporter-full.json     # Node Exporter full dashboard
│   │       └── datasources
│   │           └── datasource.yml  # Grafana datasource configuration
│   └── prometheus
│       └── prometheus.yml           # Prometheus configuration file
```

## Files Overview

### `logging/loki/loki-config.yaml`
This file configures **Loki**, a log aggregation system, to store and manage logs collected from Promtail across the control and agent servers. Loki is integrated with Grafana for visualizing logs.

### `logging/promtail/agents-server/promtail-config.yml`
This configuration file defines how **Promtail** collects logs from Docker containers and system logs on remote agent servers. Promtail forwards the logs to the Loki server running on the control server.

### `logging/promtail/control-server/promtail-config.yml`
This configuration file defines how **Promtail** collects logs on the control server and forwards them to the local Loki instance.

### Monitoring and Logging Components

#### **Prometheus**
Collects metrics from `node-exporter` and `cAdvisor` running on the control and agent servers.

#### **Grafana**
Visualizes metrics from Prometheus and logs from Loki. It provides predefined dashboards for both monitoring and logging.

#### **Loki**
A log aggregation system that stores and manages logs collected by Promtail. Loki integrates with Grafana for querying and visualization.

#### **Promtail**
A log shipping agent that forwards logs to Loki. It is deployed on both the control server and agent servers.

### Deployment Instructions

#### 1. Setup on the Control Server (STG01)

1. Clone the repository configurations to each remote server:
   ```bash
   git clone https://github.com/iAlexeze/docker-multi-server-monitoring.git logging-and-monitoring
   ```
2. Navigate to the project directory:
   ```bash
   cd logging-and-monitoring
   ```
3. **Update Prometheus Configuration**:  
   Before starting the services, update `prometheus/prometheus.yml` with the appropriate IPs of the agent servers. Example:

   ```yaml
   scrape_configs:
     - job_name: 'agents-cadvisor'
       scrape_interval: 5s
       static_configs:
         - targets:
             - 'STG02_IP:5003'   # cAdvisor on STG02
             - 'DEV01_IP:5003'   # cAdvisor on DEV01
             - 'DEV02_IP:5003'   # cAdvisor on DEV02
             - 'DEV03_IP:5003'   # cAdvisor on DEV03

     - job_name: 'agents-node-exporter'
       scrape_interval: 5s
       static_configs:
         - targets:
             - 'STG02_IP:5004'   # Node Exporter on STG02
             - 'DEV01_IP:5004'   # Node Exporter on DEV01
             - 'DEV02_IP:5004'   # Node Exporter on DEV02
             - 'DEV03_IP:5004'   # Node Exporter on DEV03
   ```
4. **Port Management**:
   Ensure that control server can accept traffic from the following ports:
   ```yaml
      - 5001: for `Grafana UI`
      - 5002: for `Loki`
   ```
5. Start the stack:
   ```bash
   docker-compose -f docker-compose-control.yml up -d
   ```
6. Access Grafana at `http://<STG01_IP>:5001`.

#### 2. Setup on Remote Servers (e.g., STG02, DEV01, DEV02, DEV03)

1. Clone the repository configurations to each remote server:
   ```bash
   git clone https://github.com/iAlexeze/docker-multi-server-monitoring.git logging-and-monitoring
   ```
2. Navigate to the project directory:
   ```bash
   cd logging-and-monitoring
   ```
3. **Update Promtail Configuration**:  
   Before starting the services, edit the `promtail-config.yml` file in the agents' directory and set the `clients` field to the control server's IP address. Example:

   ```yaml
   clients:
     - url: http://<control_server_ip>:5002/loki/api/v1/push
   ```
4. **Port Management**:
   Ensure that remote servers can accept traffic from the following ports:
   ```yaml
      - 5003: for `cAdvisor`
      - 5004: for `Node-Exporter`
   ```
5. Start the agents:
   ```bash
   docker-compose -f docker-compose-agents.yml up -d
   ```

#### 3. Automate Setup with Ansible

To streamline the deployment process on remote servers, we use **Ansible** for automation. Here's an example of a simple playbook outline:

1. **Install Ansible** on your control machine (if not already installed):
   ```bash
   sudo apt install ansible -y
   ```

2. **Create an Ansible inventory file (`inventory.yml`)**:
   ```yaml
   all:
     hosts:
       stg02:
         ansible_host: <STG02_IP>
       dev01:
         ansible_host: <DEV01_IP>
       dev02:
         ansible_host: <DEV02_IP>
       dev03:
         ansible_host: <DEV03_IP>
   ```

3. **Write an Ansible playbook (`deploy.yml`)**:
   ```yaml
   - name: Deploy Monitoring Agents
     hosts: all
     tasks:
       - name: Clone the repository
         git:
           repo: 'https://github.com/iAlexeze/docker-multi-server-monitoring.git'
           dest: '/path/to/deployment'
           update: yes

       - name: Start the Docker Compose agents
         command: docker-compose -f /path/to/deployment/docker-compose-agents.yml up -d
   ```

4. **Run the playbook**:
   ```bash
   ansible-playbook -i inventory.yml deploy.yml
   ```

#### 3. Configure Loki in Grafana

- Use the **Loki** datasource for querying and visualizing logs.

#### 4. View Logs in Grafana

1. Navigate to the "Explore" section in Grafana.
2. Choose the Loki datasource to view logs.
