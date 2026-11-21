# Slate AX Toolset

Turns a GL.iNet Slate AX (GL-AXT1800, OpenWrt) into an AV programming and commissioning tool.

| # | Piece | Status |
|---|---|---|
| 1 | [Vendor-based DHCP ranges](01-vendor-dhcp/) (Crestron in one range, audio in another...) | Built, tested on simulated network; needs on-device check |
| 2 | Live inventory page: planned vs. actual IPs, CSV as-built export | Next |
| 3 | Discovery listeners (Crestron autodiscovery, Dante mDNS, PJLink, SSDP, LLDP) for static-IP gear | Planned |
| 4 | Link-local (169.254) catcher | Planned |
| 5 | "Which switch port am I on?" via LLDP | Planned |
| 6 | Status grid: ping + key-port checks per device | Planned |
| 7 | Site services: NTP, syslog, TFTP, rogue-DHCP alert, packet capture | Planned |
| 8 | Remote access: WireGuard/Tailscale, ser2net for RS-232, Wake-on-LAN | Planned |
