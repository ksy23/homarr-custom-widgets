# Homarr v2 Custom VPN Widgets (Prometheus)

A collection of custom table widgets for **Homarr v2** built using the `homarr-custom-widget-v2` schema. These widgets query **Prometheus** directly to display live peer and device status for **WireGuard** (including PiVPN and multi-server setups) and **Tailscale** networks.

## Repository Structure

```text
.
├── README.md
├── tailscale/
│   ├── README.md
│   └── tailscale-devices-info-widget.json
└── wireguard/
    ├── README.md
    └── wireguard-peers-widget.json
```

---

## Included Widgets

### 1. [WireGuard Peers (Multi-Server)](./wireguard/README.md)
* **File:** [`wireguard-list/wireguard_widget.json`](./wireguard-list/wireguard_widget.json)
* **Exporter:** [`mindflavor/prometheus-wireguard-exporter`](https://github.com/mindflavor/prometheus_wireguard_exporter)
* **Columns Displayed:** `server`, `friendly_name` (PiVPN client names), `ip`, `received_bytes`, `sent_bytes`, and color-coded `latest_handshake` badges.
* **Highlights:** Supports scraping multiple WireGuard servers into a single table, formats Rx/Tx traffic automatically (`B`–`GB`), and includes live + persistent sorting at a readable `14px` font size.

### 2. [Tailscale Devices Info](./tailscale-device-info-list/README.md)
* **File:** [`tailscale-device-info-list/tailscale_devices_info.json`](./tailscale-device-info-list/tailscale_devices_info.json)
* **Exporter:** [`adinhodovic/tailscale-exporter`](https://github.com/adinhodovic/tailscale-exporter)
* **Columns Displayed:** `hostname`, `name`, `client_version`, `os`, `tailscale_ip`, and color-coded `last_seen` badges.
* **Highlights:** Joins `tailscale_devices_info` with `tailscale_devices_last_seen` via PromQL, uses natural numeric sorting for IPs and versions, and provides live + persistent sorting controls at `14px` font size.

---

## Quick Start

1. **Set Up Exporters & Prometheus:** Follow the exporter and `prometheus.yml` scrape configuration steps inside each widget's folder ([WireGuard Guide](./wireguard-list/README.md) | [Tailscale Guide](./tailscale-device-info-list/README.md)).
2. **Import into Homarr v2:**
   * Open Homarr and go to **Management → Custom Widgets**.
   * Click **Import** and upload the desired `.json` file.
3. **Configure Prometheus Source:**
   * On the Custom Widgets list, click **Actions (⋮) → Configure sources** on the imported widget.
   * Update the default `http://prometheus:9090` URL to match your Prometheus instance (and add authentication if enabled).
4. **Add to Your Dashboard:**
   * Enter **Edit Mode** on your Homarr board, click **Add item → Widget**, and select your imported widget.
   * Open the widget's settings while in Edit Mode if you want to customize the **Default Sort Column** or **Default Sort Order** that persists across page refreshes.

---

## Common Requirements

* **Homarr:** v2.0 or newer
* **Prometheus:** v2.x or v3.x accessible from your Homarr container/host over HTTP(S)
