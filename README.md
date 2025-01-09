# Multi-Server Monitoring with Prometheus, Grafana, cAdvisor, and Node Exporter

This project sets up a monitoring stack using **Prometheus**, **Grafana**, **cAdvisor**, and **Node Exporter** to collect and visualize metrics from multiple servers and containers. The stack is deployed using **Docker Compose** and includes configurations for monitoring Docker containers as well as system-level metrics from multiple remote servers.

## Project Structure

The project is organized as follows:

```
.
├── LICENSE
├── README.md
├── docker-compose-agents.yml       # Docker Compose configuration for agent servers
├── docker-compose-main.yml         # Docker Compose configuration for the main server (STG01)
├── grafana
│   └── provisioning
│       ├── dashboards
│       │   ├── dashboard.yml         # Grafana dashboard configuration
│       │   ├── docker-host-monitoring.json  # Docker host monitoring dashboard
│       │   └── node-exporter-full.json     # Node Exporter full dashboard
│       └── datasources
│           └── datasource.yml        # Grafana datasource configuration
└── prometheus
    └── prometheus.yml               # Prometheus configuration file
```

## Files Overview

### `LICENSE`
This file contains the licensing information for the project.

### `README.md`
This file contains the documentation for the project, describing its purpose and structure.

### `docker-compose-agents.yml`
This Docker Compose file is used for deploying **node-exporter** and **cAdvisor** on **remote servers** (e.g., STG02, DEV01, DEV02, DEV03). These agents collect system and Docker container metrics from the remote machines and expose them for scraping by Prometheus.

- **node-exporter**: Collects system-level metrics (e.g., CPU usage, memory usage, disk I/O, network stats) from the agent server.
- **cAdvisor**: Collects container-level metrics, such as CPU, memory, and network stats from the containers running on the agent servers.

### `docker-compose-main.yml`
This Docker Compose file is used for deploying the **Prometheus** and **Grafana** services on the **main server** (STG01).

- **Prometheus**: Scrapes metrics from `node-exporter` and `cAdvisor` running on the agent servers. It also stores the metrics for later querying.
- **Grafana**: Provides a web-based UI for visualizing the data collected by Prometheus. It uses predefined dashboards to display system and container metrics.

### `grafana/provisioning/dashboards/`
Contains JSON dashboard configurations for **Grafana**.

- **dashboard.yml**: Defines the dashboards available in Grafana. It points to the JSON files in this directory for the actual dashboard configurations.
- **docker-host-monitoring.json**: A Grafana dashboard for monitoring the Docker host (system-level metrics).
- **node-exporter-full.json**: A Grafana dashboard for monitoring detailed metrics from `node-exporter` (system-level metrics).

### `grafana/provisioning/datasources/`
Contains the configuration for the **Grafana datasource** that connects Grafana to Prometheus.

- **datasource.yml**: Configures Prometheus as the data source for Grafana, specifying the URL for the Prometheus server.

### `prometheus/prometheus.yml`
This is the **Prometheus configuration file**. It defines how Prometheus scrapes metrics from various targets, including the remote agents and the local cAdvisor and Node Exporter instances. The configuration includes:

- **scrape_configs**: Defines the scraping rules for different targets (e.g., `node-exporter` on agent servers, `cAdvisor` on both main and agent servers).
- **relabel_configs**: Optionally adds labels to the scraped metrics to help differentiate between instances (e.g., by instance hostname or IP address).

## Deployment Instructions

### 1. Setup on the Main Server (STG01)

1. Copy `docker-compose-main.yml` to the main server (STG01).
2. Customize the environment variables in the `.env` file if needed (e.g., container names, image names).
3. Run the following command to start the monitoring stack:
   ```bash
   docker-compose -f docker-compose-main.yml up -d
   ```
4. Access Grafana by visiting `http://<STG01_IP>:5001` in your web browser.

### 2. Setup on Remote Servers (e.g., STG02, DEV01, DEV02, DEV03)

1. Copy `docker-compose-agents.yml` to each remote server.
2. Customize the environment variables in the `.env` file for each agent server (e.g., `NODE_EXPORTER_CONTAINER_NAME`, `CADVISOR_CONTAINER_NAME`).
3. Run the following command on each agent server:
   ```bash
   docker-compose -f docker-compose-agents.yml up -d
   ```

### 3. Configure Prometheus to Scrape Remote Agents

Make sure the **Prometheus** configuration (`prometheus.yml`) on the main server is set up to scrape the remote agent servers. Add the IP addresses of each agent server under the `scrape_configs` section:

```yaml
scrape_configs:
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['<STG02_IP>:9100', '<DEV01_IP>:9100', '<DEV02_IP>:9100', '<DEV03_IP>:9100']
```

### 4. Access Grafana Dashboards

1. Log in to Grafana at `http://<STG01_IP>:5001`.
2. Navigate to the predefined dashboards that are automatically imported, such as the **Docker Host Monitoring** and **Node Exporter Full** dashboards.

## Conclusion

This setup provides a complete monitoring solution for multiple servers and Docker containers using **Prometheus**, **Grafana**, **cAdvisor**, and **Node Exporter**. By deploying the stack across multiple servers, you can monitor both container metrics and system-level metrics efficiently.

---
