# avdhcp: DHCP ranges by manufacturer

Gives out DHCP addresses from a different range depending on who made the device
(matched by MAC OUI). Crestron gets .100–.150, audio gear gets .151–.200, and so on.
Uses the router's own dnsmasq, so nothing extra is installed.

Files:
- `avdhcp`: the script (busybox sh). Goes in `/usr/bin/`.
- `vendors.conf`: vendor key → MAC prefixes, generated from the IEEE registry. Goes in `/etc/avdhcp/`.
- `ranges.conf`: your groups and ranges, edited per job. Goes in `/etc/avdhcp/`.
- `tools/build-vendors.py`: rebuilds `vendors.conf` on your laptop.

## Install (GL.iNet Slate AX)

1. **Set the LAN subnet** in the GL admin page: *Network → LAN → Router IP* `172.22.0.1`
   (or whatever matches your ranges). Reconnect at the new address.
2. **Copy the files.** GL firmware runs dropbear without SFTP, so use `scp -O`:

```bash
ssh root@172.22.0.1 mkdir -p /etc/avdhcp
```
```bash
scp -O vendors.conf ranges.conf root@172.22.0.1:/etc/avdhcp/
```
```bash
scp -O avdhcp root@172.22.0.1:/usr/bin/
```
```bash
ssh root@172.22.0.1 chmod +x /usr/bin/avdhcp
```

3. **Preview, then apply** (on the router):

```bash
avdhcp generate
```
```bash
avdhcp apply
```

4. **Keep it through firmware upgrades:**

```bash
printf '/etc/avdhcp/\n/usr/bin/avdhcp\n/etc/dnsmasq.conf\n' >> /etc/sysupgrade.conf
```

## Daily use

| Command | What it does |
|---|---|
| `avdhcp leases` | Every lease: IP, MAC, vendor, group. Flags devices sitting in the wrong range |
| `avdhcp vendor 00:10:7F:12:34:56` | Which vendor/group a MAC maps to |
| `avdhcp apply` | Re-run after editing `ranges.conf` or `vendors.conf` |
| `avdhcp revert` | Back to stock GL DHCP |

To edit ranges on the router, use `vi /etc/avdhcp/ranges.conf` and then run `avdhcp apply`.

## How it works

- Writes a marked block into `/etc/dnsmasq.conf`, which OpenWrt's dnsmasq reads in addition
  to its own config: `dhcp-mac` tags per vendor prefix, one `dhcp-range` per group, plus a
  `default` range for everyone else.
- Sets OpenWrt's own LAN pool to **static-only** (`dhcp.lan.dynamicdhcp=0`) so it stops
  handing addresses to everyone. Reservations made in the GL/LuCI UI still work and take
  priority. `revert` puts the old setting back.
- Some vendors own a 28-bit (MA-M) or 36-bit (MA-S) block rather than a full /24 (Shure
  `9C431EE`, Biamp `34E1D19`). Those expand to exact matches, so the other companies that share
  the prefix aren't pulled in.

## Gotchas

- **Devices with an existing lease** in the wrong range move when they renew (OpenWrt's
  dnsmasq is authoritative, so it NAKs them quickly). Power-cycle a device to move it right away.
  `avdhcp leases` shows any device that hasn't moved yet.
- **Rebranded / OEM gear:** a device built on a third-party network module shows that module
  maker's OUI. Look up the MAC with `avdhcp vendor`, then add its prefix to the right vendor
  line in `vendors.conf`. Dante gear from many brands shows up as `audinate`.
- **Static-IP devices** never ask for DHCP. Discovery (a later piece of this toolkit) is what finds those.
- **Saving the LAN page in the GL UI** may rewrite the DHCP settings. If devices start landing
  in the wrong place afterwards, run `avdhcp apply` again.
- If `apply` warns that dnsmasq's generated config doesn't include `/etc/dnsmasq.conf`, tell me.
  It means this firmware build loads config differently and the script needs a tweak.

## Refreshing the vendor list

```bash
cd tools && curl -O https://standards-oui.ieee.org/oui/oui.csv -O https://standards-oui.ieee.org/oui28/mam.csv -O https://standards-oui.ieee.org/oui36/oui36.csv
```
```bash
python3 build-vendors.py oui.csv mam.csv oui36.csv > ../vendors.conf
```

To add a manufacturer, add a line to the `VENDORS` table at the top of `build-vendors.py`.
