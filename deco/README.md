# Homarr v2 TP-Link Deco Widgets (Prometheus)

A suite of three custom widgets for **Homarr v2** built using the `homarr-custom-widget-v2` schema to monitor **TP-Link Deco** mesh routers (tested on **Deco BE11000** Wi-Fi 7) via **Prometheus** and [`SimonJPegg/deco-rate`](https://github.com).

## Included Widgets

| Widget | File | Purpose |
| :--- | :--- | :--- |
| **1. Internet & Bandwidth Overview** | [`deco-internet-overview-widget.json`](./deco-internet-overview-widget.json) | Dynamically displays your Deco model name, online status, live aggregate Download/Upload speeds (`MB/s` + `Mbps`) with 30-minute background sparkline charts, total connected clients, online mesh nodes, CPU/Memory load, and firmware update status. |
| **2. Connected Clients Table** | [`deco-clients-table-widget.json`](./deco-clients-table-widget.json) | Displays every connected client's `name`, `ip`, color-coded Wi-Fi `band` (`6 GHz`, `5 GHz`, `2.4 GHz`), live `rx_speed`, live `tx_speed`, and `status` (`Online`/`Offline`) at `14px` font with live + persistent sorting. |
| **3. Mesh Nodes Health Table** | [`deco-nodes-health-widget.json`](./deco-nodes-health-widget.json) | Displays each Deco unit's `name`, `role` (`Main` vs. `Satellite`), `ip`, `backhaul` bands, per-node `clients` count, `signal (2G / 5G)` strength, and `status` at `14px` font with live + persistent sorting. |

---

## Prerequisites

1. **Homarr v2** (with Custom Widgets v2 support).
2. **Prometheus** server reachable from your Homarr instance.
3. **[SimonJPegg/deco-rate](https://github.com)** (`ghcr.io/simonjpegg/deco-rate:0.0.5`) polling your main Deco node.

---

## Step 1: Set Up `deco-rate` Exporter

### 1. Create `config.yaml`
Create a `config.yaml` file using your main Deco's gateway IP and your **TP-Link ID email and password**:

```yaml
host: "http://192.168.68.1" # Replace with your main Deco IP (e.g., 192.168.1.1 or 192.168.68.1)
username: "your-tplink-email@example.com"
password: "your-tplink-password"
poll_interval_seconds: 30
metrics_port: 9550
verify_ssl: false
timeout_seconds: 30
```

*(Note: Keep `poll_interval_seconds` at `30` or higher so you don't overwhelm the Deco's lightweight local web server.)*

### 2. Deploy via Docker Compose
Use the tagged `0.0.5` multi-arch image (`linux/amd64` and `linux/arm64` for Raspberry Pi):

```yaml
services:
  deco_rate:
    image: ghcr.io/simonjpegg/deco-rate:0.0.5
    container_name: deco_rate
    ports:
      - "9550:9550"
    volumes:
      - ./config.yaml:/etc/deco-rate/config.yaml:ro
    restart: unless-stopped
```

Verify metrics are exposed on port `9550`:
```bash
curl -s http://localhost:9550/metrics | grep ^deco_
```

---

## Step 2: Configure Prometheus Scrape Target

Add the `deco` scrape job to your `prometheus.yml` and reload or restart Prometheus:

```yaml
scrape_configs:
  - job_name: 'deco'
    scrape_interval: 30s
    static_configs:
      - targets: ['<docker-host-ip>:9550']
```

---

## Step 3: Import Widgets into Homarr v2

1. Open Homarr and go to **Management → Custom Widgets**.
2. Click **Import** and upload each of the three `.json` files:
   * `deco-internet-overview-widget.json`
   * `deco-clients-table-widget.json`
   * `deco-nodes-health-widget.json`
3. For each imported widget, click **Actions (⋮) → Configure sources** and update `http://prometheus:9090` to your Prometheus server URL.
4. Enter **Edit Mode** on your Homarr dashboard, click **Add item → Widget**, and place the widgets on your board.

---

## Widget Options (Persistent Settings)

For the **Connected Clients Table** and **Mesh Nodes Health Table**, you can use the dropdowns at the top of the widget for temporary sorting, or open the widget's settings while in board **Edit Mode** to save persistent defaults across page reloads:

| Widget | Option | Valid Values | Default |
| :--- | :--- | :--- | :--- |
| **Connected Clients** | `Default Sort Column` | `name`, `ip`, `connection_type`, `rx_speed`, `tx_speed`, `status` | `rx_speed` |
| **Connected Clients** | `Default Sort Order` | `asc`, `desc` | `desc` |
| **Mesh Nodes** | `Default Sort Column` | `name`, `role`, `ip`, `clients`, `status` | `role` |
| **Mesh Nodes** | `Default Sort Order` | `asc`, `desc` | `asc` |
