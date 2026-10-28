<div align="center">

<img src="snap/gui/ruview.png" alt="RuView logo" width="128" />

# RuView

**WiFi sensing, a local web dashboard and sensor tools.**

[![ruview](https://snapcraft.io/ruview/badge.svg)](https://snapcraft.io/ruview)
[![Base: core26](https://img.shields.io/badge/Base-core26-2C8C3C?logo=canonical&logoColor=white)](https://snapcraft.io/core26)
![Confinement: strict](https://img.shields.io/badge/Confinement-strict-2C8C3C)
[![Maintainer: sidaliteir](https://img.shields.io/badge/Maintainer-sidaliteir-2C8C3C?logo=snapcraft&logoColor=white)](https://snapcraft.io/publisher/sidaliteir)
![Architectures: amd64 and arm64](https://img.shields.io/badge/Architectures-amd64%20%7C%20arm64-2C8C3C?logo=linux&logoColor=white)

[Getting Started](#getting-started) · [Configure](#configure) · [Limitations](#limitations) · [Build](#build) · [Support](#support) · [License](#license)

Unofficial snap packaging for [RuView](https://github.com/ruvnet/RuView), including the sensing server, browser dashboard and operator toolkit.

</div>

---

## Getting Started

<a href="https://snapcraft.io/ruview"><img alt="Get it from the Snap Store" src="https://snapcraft.io/en/dark/install.svg" width="180"></a>

```bash
sudo snap install ruview
```

Open [http://127.0.0.1:8080/ui/index.html](http://127.0.0.1:8080/ui/index.html). The service starts automatically with the ESP32 data source selected. It waits for sensor packets; installation alone does not provide real sensing data.

Real sensing requires compatible ESP32 hardware running the [upstream CSI firmware](https://github.com/ruvnet/RuView/tree/main/firmware/esp32-csi-node). Configure the node to send packets to this machine on UDP port `5005`, then allow its address:

```bash
sudo snap set ruview udp-address=0.0.0.0 udp-allow=192.168.1.50
```

Replace `192.168.1.50` with your sensor's IP address. The toolkit can flash a downloaded firmware bundle, and `ruview.provision` configures the node's WiFi and server address.

To explore the dashboard with synthetic data:

```bash
sudo snap set ruview source=simulated
```

Simulated output is a demonstration, not a measurement of people or a room. Restore sensor input with `sudo snap set ruview source=esp32`.

| Command | Purpose |
| --- | --- |
| `ruview` | Run the sensing server in the foreground |
| `ruview.toolkit` | Diagnostics, device tools, firmware flashing and MCP |
| `ruview.cli` | `wifi-densepose` calibration, enrollment and room commands |
| `ruview.provision` | Configure ESP32 WiFi and sensor settings |

```bash
ruview.toolkit --help
ruview.toolkit doctor --url http://127.0.0.1:8080
ruview.toolkit mcp start
```

Node, Python, esptool, pyserial, the NVS generator and the WASM sensing kernel are included. For USB sensors:

```bash
sudo snap connect ruview:raw-usb
```

For other UART devices, use `snap interface serial-port` to find the device slot, then `sudo snap connect ruview:serial-port PROVIDER:SLOT`.

For remote tools, connect `sudo snap connect ruview:ssh-keys` and establish host trust with SSH first. Remote hosts need Node/npm because the toolkit runs through `npx` there. Upstream supports file-based keys here, not SSH-agent authentication.

## Configure

| Setting | Accepted values | Default |
| --- | --- | --- |
| `source` | `esp32`, `simulated` | `esp32` |
| `address` | HTTP and WebSocket IPv4 listen address, without a port | `127.0.0.1` |
| `allowed-hosts` | Comma-separated dashboard host names or IP addresses | Unset |
| `api-token` | API access token, without whitespace | Unset |
| `udp-address` | Sensor UDP IPv4 listen address, without a port | `127.0.0.1` |
| `udp-allow` | Comma-separated sensor IPv4 addresses or CIDR ranges | Unset |

```bash
sudo snap get ruview
sudo snap services ruview
sudo snap logs ruview -f
```

The service uses HTTP port `8080`, WebSocket port `8765`, and UDP port `5005`. Network listeners default to this machine only. Changes restart the running service.

For dashboard access over a trusted LAN, set `address=0.0.0.0`, a strong `api-token`, and `allowed-hosts` to this machine's LAN address in the same `snap set` command. The dashboard uses HTTP; use a TLS reverse proxy for access over untrusted networks.

Service data persists in `/var/snap/ruview/common/data`. To run in the foreground, stop the service first to free its ports:

```bash
sudo snap stop ruview.daemon
ruview --help
ruview --source simulated
```

Use `ruview --source esp32` for foreground sensor input. Foreground data is stored separately in `~/snap/ruview/common/data`. Stop the foreground process before restarting the service with `sudo snap start ruview.daemon`.

Toolkit capture and CLI calibration also listen on UDP `5005`. Stop the service before using them on that port, then restart it afterwards. Toolkit settings and credentials are stored under the snap's user home, `~/snap/ruview/current`.

## Limitations

- ESP32 hardware and firmware are not included. The snap receives packets from separately configured sensors over the network.
- Capturing CSI directly from the Linux host's WiFi adapter is unavailable. Use networked ESP32 sensors or explicitly select the synthetic demonstration.
- Synthetic data does not establish sensing accuracy. Real sensing depends on the sensors, firmware, environment, and upstream model support.
- Desktop and mobile applications are not bundled. Full model training and proof replay require the upstream source workspace and their development dependencies.
- The CLI's MAT disaster assessment feature is not included. Use `ruview.cli train-room` for room training.
- The browser needs internet access for upstream CDN assets. Optional browser WASM acceleration uses the upstream JavaScript fallback.

## Build

```bash
git clone https://github.com/sidalit/ruview-snap.git
cd ruview-snap
sudo snap install snapcraft --classic
snapcraft pack
```

Versions follow upstream's numeric snapshots (`v3422`, etc.); the bundled toolkit has its own version.

## Support

- **Installation or snap issues:** [open a packaging issue](https://github.com/sidalit/ruview-snap/issues).
- **Application bugs:** [report them upstream](https://github.com/ruvnet/RuView/issues).

## License

Snap packaging: [MIT](https://github.com/sidalit/ruview-snap/blob/main/LICENSE).
