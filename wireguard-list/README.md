# Homarr v2 WireGuard Peers Widget (Prometheus)

A custom widget for **Homarr v2** that displays real-time WireGuard peer status, friendly client names (from PiVPN or `wg0.conf`), IP addresses, Rx/Tx bandwidth usage, and color-coded handshake timestamps across one or multiple WireGuard servers via Prometheus.

## Features

* **Multi-Server Support:** Distinguishes peers across multiple WireGuard instances using custom Prometheus `server` labels (with automatic fallback to the exporter's `instance` IP).
* **PiVPN Friendly Names:** Automatically resolves human-readable peer names instead of raw public keys.
* **Color-Coded Handshake Badges:**
  * 🟢 **Green (`< 3m ago`):** Actively connected
  * 🟡 **Yellow (`3m – 1h ago`):** Recently active / idle
  * ⚪ **Gray (`> 1h ago` or `Never`):** Offline or never connected
* **Human-Readable Bandwidth:** Automatically formats `received_bytes` and `sent_bytes` into `B`, `KB`, `MB`, or `GB`.
* **Live & Persistent Sorting:** Includes header dropdowns for on-the-fly sorting (`server`, `friendly_name`, `ip`, `received_bytes`, `sent_bytes`, `latest_handshake`) plus persistent widget options that survive page refreshes.
* **Clean `14px` Table Layout:** Scrollable X/Y table optimized for readability.

---

## Prerequisites

1. **Homarr v2** (with Custom Widgets v2 support).
2. **Prometheus** server reachable from your Homarr instance.
3. **[mindflavor/prometheus-wireguard-exporter](https://github.com/MindFlavor/prometheus_wireguard_exporter)** running on each WireGuard host.

---

## Step 1: Configure `prometheus-wireguard-exporter`

### 1. Add `# friendly_name` Comments for PiVPN
Newer versions of `prometheus-wireguard-exporter` (v3.5+) read `# friendly_name = <name>` comments rather than PiVPN's `### begin <name> ###` blocks. Run this command on your PiVPN host to inject compatible comments without breaking `pivpn` CLI functionality:

```bash
sudo sed -i -E 's/^### begin (.+) ###\$/### begin \1 ###\n# friendly_name = \1/' /etc/wireguard/wg0.conf
```

*(Note: Re-run this command and restart the exporter container whenever you add a new client with `pivpn -a`.)*

### 2. Deploy via Docker Compose
Run the exporter with `user: "0:0"`, `network_mode: host`, and `NET_ADMIN` capability so it can access the host `wg0` interface:

```yaml
services:
  wireguard_exporter:
    image: mindflavor/prometheus-wireguard-exporter
    container_name: wireguard_exporter
    user: "0:0"
    command:
      - '--extract_names_config_files=/etc/wireguard/wg0.conf'
    network_mode: host
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
    volumes:
      - '/etc/wireguard/wg0.conf:/etc/wireguard/wg0.conf:ro'
```

Verify metrics are exposed on port `9586`:
```bash
curl -s http://localhost:9586/metrics | grep friendly_name
```

---

## Step 2: Configure Prometheus Scrape Targets

Add your WireGuard exporter target(s) to `prometheus.yml`. Use the `labels.server` field to give each server a clean display name in the widget:

```yaml
scrape_configs:
  - job_name: 'wireguard'
    static_configs:
      - targets: ['192.168.1.10:9586']
        labels:
          server: 'Home PiVPN'
      - targets: ['10.0.0.5:9586']
        labels:
          server: 'Cloud VPS'
```

Reload or restart Prometheus to begin scraping.

---

## Step 3: Import into Homarr v2

1. Open Homarr and navigate to **Management → Custom Widgets**.
2. Click **Import** and upload `wireguard-peers-widget.json`.
3. On the Custom Widgets list, click **Actions (⋮) → Configure sources** on **WireGuard Peers (Multi-Server)** and update `http://prometheus:9090` to your Prometheus server URL.
4. Go to your dashboard, enter **Edit Mode**, click **Add item → Widget**, and select **WireGuard Peers (Multi-Server)**.

---

## Widget Options (Persistent Settings)

Open the widget's settings while in board **Edit Mode** to configure defaults that persist across page reloads:

| Option | Description | Default |
| :--- | :--- | :--- |
| **Default Sort Column** | Column to sort by on load (`server`, `friendly_name`, `ip`, `received_bytes`, `sent_bytes`, `latest_handshake`) | `latest_handshake` |
| **Default Sort Order** | Sort direction on load (`asc` or `desc`) | `desc` |
