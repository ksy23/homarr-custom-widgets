# Homarr v2 Custom VPN Widgets (Prometheus)

A collection of custom table widgets for **Homarr v2** built using the `homarr-custom-widget-v2` schema. These widgets query **Prometheus** directly to display live peer and device status for **WireGuard** (including PiVPN and multi-server setups) and **Tailscale** networks.

## Repository Structure

```text
.
├── README.md
├── deco/
│   ├── README.md
│   ├── deco-internet-overview-widget.json
│   ├── deco-clients-table-widget.json
│   └── deco-nodes-health-widget.json
│   ├── deco-internet-overview-widget.png
│   ├── deco-clients-table-widget.png
│   └── deco-nodes-health-widget.png
├── tailscale/
│   ├── README.md
│   └── tailscale-devices-info-widget.json
│   └── tailscale-devices-info-widget.png
└── wireguard/
    ├── README.md
    └── wireguard-peers-widget.json
    └── wireguard-peers-widget.png
```

---

## Included Widgets

### 1. [WireGuard Peers (Multi-Server)](./wireguard/README.md)
* **File:** [`wireguard/wireguard-peers-widget.json`](./wireguard/wireguard-peers-widget.json)
* **Exporter:** [`mindflavor/prometheus-wireguard-exporter`](https://github.com/mindflavor/prometheus_wireguard_exporter)
* **Columns Displayed:** `server`, `friendly_name` (PiVPN client names), `ip`, `received_bytes`, `sent_bytes`, and color-coded `latest_handshake` badges.
* **Highlights:** Supports scraping multiple WireGuard servers into a single table, formats Rx/Tx traffic automatically (`B`–`GB`), and includes live + persistent sorting at a readable `14px` font size.

### 2. [Tailscale Devices Info](./tailscale/README.md)
* **File:** [`tailscale/tailscale-devices-info-widget.json`](./tailscale/tailscale-devices-info-widget.json)
* **Exporter:** [`adinhodovic/tailscale-exporter`](https://github.com/adinhodovic/tailscale-exporter)
* **Columns Displayed:** `hostname`, `name`, `client_version`, `os`, `tailscale_ip`, and color-coded `last_seen` badges.
* **Highlights:** Joins `tailscale_devices_info` with `tailscale_devices_last_seen` via PromQL, uses natural numeric sorting for IPs and versions, and provides live + persistent sorting controls at `14px` font size.

### 3. [TP-Link Deco Mesh Suite (3 Widgets)](./deco/README.md)
* **Exporter:** [`SimonJPegg/deco-rate`](https://github.com/SimonJPegg/deco-rate) (`ghcr.io/simonjpegg/deco-rate:0.0.5`)
* **Widgets:**
  * **[`deco-internet-overview-widget.json`](./deco/deco-internet-overview-widget.json):** Dynamic Deco model header, online status, live aggregate Download/Upload throughput (`MB/s` + `Mbps`) with 30-minute background sparkline charts, total clients, online mesh nodes, CPU/Memory %, and firmware status.
  * **[`deco-clients-table-widget.json`](./deco/deco-clients-table-widget.json):** Connected client `name`, `ip`, color-coded Wi-Fi `band` (`6 GHz`, `5 GHz`, `2.4 GHz`), live `rx_speed`, live `tx_speed`, and `status` at `14px` font with live + persistent sorting.
  * **[`deco-nodes-health-widget.json`](./deco/deco-nodes-health-widget.json):** Mesh node `name`, `role` (`Main` vs. `Satellite`), `ip`, `backhaul` bands, per-node `clients` count, `signal (2G / 5G)` levels, and `status` at `14px` font with live + persistent sorting.
---

## Quick Start

1. **Set Up Exporters & Prometheus:** Follow the exporter and `prometheus.yml` scrape configuration steps inside each widget's folder ([TP-Link Deco Guide](./deco/README.md) | [WireGuard Guide](./wireguard/README.md) | [Tailscale Guide](./tailscale/README.md)).
2. **Import into Homarr v2:**
   * Open Homarr and go to **Management → Custom Widgets**.
   * Click **Import** and upload the desired `.json` file(s).
3. **Configure Prometheus Source:**
   * On the Custom Widgets list, click **Actions (⋮) → Configure sources** on each imported widget.
   * Update the default `http://prometheus:9090` URL to match your Prometheus instance (and add authentication if enabled).
4. **Add to Your Dashboard:**
   * Enter **Edit Mode** on your Homarr board, click **Add item → Widget**, and select your imported widget(s).
   * Open any table widget's settings while in Edit Mode to customize the **Default Sort Column** or **Default Sort Order** that persists across page refreshes.

---

## Common Requirements

* **Homarr:** v2.0 or newer
* **Prometheus:** v2.x or v3.x accessible from your Homarr container/host over HTTP(S)
