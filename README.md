<div align="center">

<img src="snap/gui/ruview.png" alt="RuView logo" width="128" />

# RuView

**WiFi sensing with a local web dashboard.**

[![ruview](https://snapcraft.io/ruview/badge.svg)](https://snapcraft.io/ruview)
[![Base: core26](https://img.shields.io/badge/Base-core26-2C8C3C?logo=canonical&logoColor=white)](https://snapcraft.io/core26)
![Confinement: strict](https://img.shields.io/badge/Confinement-strict-2C8C3C)
[![Maintainer: sidaliteir](https://img.shields.io/badge/Maintainer-sidaliteir-2C8C3C?logo=snapcraft&logoColor=white)](https://snapcraft.io/publisher/sidaliteir)
![Architectures: amd64 and arm64](https://img.shields.io/badge/Architectures-amd64%20%7C%20arm64-2C8C3C?logo=linux&logoColor=white)

[Getting Started](#getting-started) · [Configure](#configure) · [Limitations](#limitations) · [Build](#build) · [Support](#support) · [License](#license)

Unofficial snap packaging for [RuView](https://github.com/ruvnet/RuView), a WiFi sensing server with a browser dashboard.

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

Replace `192.168.1.50` with your sensor's IP address. Hardware setup and firmware flashing are performed separately from the snap.

To explore the dashboard with synthetic data:

```bash
sudo snap set ruview source=simulated
```

Simulated output is a demonstration, not a measurement of people or a room. Restore sensor input with `sudo snap set ruview source=esp32`.

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

## Limitations

- ESP32 hardware and firmware are not included. The snap receives packets from separately configured sensors over the network.
- Capturing CSI directly from the Linux host's WiFi adapter is unavailable. Use networked ESP32 sensors or explicitly select the synthetic demonstration.
- Synthetic data does not establish sensing accuracy. Real sensing depends on the sensors, firmware, environment, and upstream model support.
- The server's CLI and web dashboard are included. Desktop applications, mobile applications, firmware flashing, and contributor harnesses are not bundled.
- The browser needs internet access for upstream CDN assets. Optional browser WASM acceleration uses the upstream JavaScript fallback.

## Build

```bash
git clone https://github.com/sidalit/ruview-snap.git
cd ruview-snap
sudo snap install snapcraft --classic
snapcraft pack
```

Versions follow upstream's numeric sensing snapshots (`v3422`, etc.), independently of its AI harness releases.

## Support

- **Installation or snap issues:** [open a packaging issue](https://github.com/sidalit/ruview-snap/issues).
- **Application bugs:** [report them upstream](https://github.com/ruvnet/RuView/issues).

## License

Snap packaging: [MIT](https://github.com/sidalit/ruview-snap/blob/main/LICENSE).
