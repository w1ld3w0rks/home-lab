# 🏠 Home Lab — UniFi Network Build

> A fully documented home network overhaul built on the UniFi ecosystem.  
> Built alongside CCNA certification studies.

---

## 📡 Network Diagram

**[▶ View Live Network Diagram](docs/network-diagram.html)**

---

## 🎯 Project Goals

- Replace consumer Verizon router with enterprise-grade UniFi stack
- Install structured Cat6A cabling throughout the home
- Provide isolated WiFi for a neighbor via wireless mesh (no cable run)
- Segment traffic using VLANs (personal, neighbor, IoT/cameras)
- Build CCNA-relevant hands-on experience with real equipment
- Document the full build for reference and portfolio

---

## 🛒 Equipment List

### Core Infrastructure

| Device | Model | Role | Price |
|--------|-------|------|-------|
| UniFi Dream Machine Pro | UDM-Pro | Gateway / Controller / Firewall | $379 |
| UniFi Switch 16 PoE | USW-16-POE | Layer 2 managed PoE switch | $249 |
| UniFi UPS 2U | UPS-2U-US | Battery backup / power protection | ~$399 |
| UniFi PDU Pro | USP-PDU-Pro | Smart power distribution (16 outlets) | ~$299 |
| SFP+ DAC Cable (0.5m) | — | 10G uplink: UDM Pro ↔ Switch | ~$15 |

### Rack & Cabling

| Item | Spec | Price |
|------|------|-------|
| NavePoint 15U Wall-Mount Rack | ~23.6" depth | ~$175 |
| Cat6A 12-Port Patch Panel x2 | Split by floor/area | ~$90 |
| Cable Management Bar x2 | 1U each | ~$30 |

### Wireless

| Device | Model | Location | Uplink |
|--------|-------|----------|--------|
| UniFi U6 Pro | U6-Pro | Upstairs Loft | Cat6A (PoE) |
| UniFi U6 Pro | U6-Pro | Kitchen | Cat6A (PoE) |
| UniFi U6 Mesh Pro | U6-Mesh-Pro | Neighbor's section | Wireless uplink |

**Estimated Total: ~$2,100–$2,150**

---

## 🗄️ Rack Layout (15U)

```
┌─────────────────────────────────────┐
│  1U  │ UniFi Dream Machine Pro       │
│  1U  │ UniFi USW-16-POE Switch       │
│  1U  │ Patch Panel A (Floor 1 runs)  │
│  1U  │ Patch Panel B (Floor 2 runs)  │
│  1U  │ Cable Management Bar x2       │
│  2U  │ UniFi UPS 2U                  │
│  2U  │ UniFi PDU Pro                 │
│──────│───────────────────────────────│
│  5U  │ FREE — Growth headroom        │
│      │ (NAS, cameras, expansion)     │
└─────────────────────────────────────┘
```

**Deepest device:** UDM Pro at 11.2" — rack must be at least 12" internal depth.  
**UPS note:** Tower form factor sits outside rack on floor/shelf.

---

## 🌐 Network Architecture

### VLAN Segmentation

| VLAN | Name | Devices | Notes |
|------|------|---------|-------|
| VLAN 10 | Work (Trusted) | Laptops, phones, NAS/backups | Full access to printers & NAS |
| VLAN 20 | Gaming (Performance) | PS5/console, AppleTV | QoS priority, lowest latency |
| VLAN 30 | IoT (Untrusted) | Cameras, smart TV, thermostat, speakers, doorbell | No access to Work devices |
| Guest | Guest WiFi | Visitor devices | Internet only, no LAN access |

### SSID Plan

| SSID | VLAN | Access |
|------|------|--------|
| WiFi-Work | VLAN 10 | Full LAN + trusted device access |
| WiFi-Game | VLAN 20 | Internet + QoS prioritized |
| WiFi-IoT | VLAN 30 | Internet only, isolated from LAN |
| Guest WiFi | — | Internet only, no LAN access |

### Connection Map

```
[Verizon ONT / Router] ──WAN──▶ [UDM Pro]
                                     │
                              SFP+ DAC (10G)
                                     │
                              [USW-16-POE]
                           ┌─────────┴──────────┐
                    Cat6A PoE            Cat6A PoE
                           │                    │
                    [U6 Pro Loft]       [U6 Pro Kitchen]
                           │
                    Wireless Uplink
                           │
                   [U6 Mesh Pro]
                  (Neighbor section)

Wired Drops:
├── Camera Run 1 (PoE · VLAN 30)
├── Camera Run 2 (PoE · VLAN 30)
├── TV / Game System (VLAN 10)
└── Loft Office (VLAN 10)
```

### Power Chain

```
[Wall Outlet] ──AC──▶ [UPS 2U] ──AC──▶ [PDU Pro] ──AC──▶ [UDM Pro + Switch]
[Wall Outlet] ──AC──▶ [Verizon Router] (temporary — removed at fiber cutover)
```

---

## 🔥 Firewall Rules

| # | Rule | Source | Destination | Action |
|---|------|--------|-------------|--------|
| 1 | Block IoT → Work | VLAN 30 | VLAN 10 | ❌ Block |
| 2 | Allow Work → IoT | VLAN 10 | VLAN 30 | ✅ Allow (if needed) |
| 3 | Allow Gaming → Internet | VLAN 20 | WAN | ✅ Allow |
| 4 | Block IoT → LAN | VLAN 30 | Any LAN | ❌ Block |
| 5 | Allow All → DNS/NTP | Any | DNS/NTP | ✅ Allow |
| 6 | Admin access | VLAN 10 | Controller/UI | ✅ Work VLAN only |

---

## 📋 Device Checklist

| Device | VLAN | Network | Connection |
|--------|------|---------|------------|
| Laptop(s) | 10 | Work (Trusted) | WiFi-Work or wired |
| Phone(s) | 10 | Work (Trusted) | WiFi-Work |
| NAS / Backups | 10 | Work (Trusted) | Wired preferred |
| PS5 / Console | 20 | Gaming (Performance) | WiFi-Game or wired |
| AppleTV | 20 | Gaming (Performance) | WiFi-Game or wired |
| UniFi G6 Dome — Cam 1 | 30 | IoT (Untrusted) | Wired PoE |
| UniFi G6 Dome — Cam 2 | 30 | IoT (Untrusted) | Wired PoE |
| Smart TV | 30 | IoT (Untrusted) | WiFi-IoT |
| Thermostat | 30 | IoT (Untrusted) | WiFi-IoT |
| Speakers | 30 | IoT (Untrusted) | WiFi-IoT |
| Doorbell | 30 | IoT (Untrusted) | WiFi-IoT |
| Guest devices | — | Guest WiFi | Internet only |

---

## 🔌 Structured Cabling

### Spec
- **Cable:** Cat6A (augmented) — supports 10GbE at full 100m
- **Termination:** 110 punch-down at keystones → wall plates → patch panels
- **Connectors:** Cat6A keystone jacks (Monoprice / Infinity Cable Products)
- **No field-terminated plugs** on in-wall runs

### Cable Runs Planned

| Run | Destination | Purpose | VLAN |
|-----|-------------|---------|------|
| 1 | Upstairs Loft | U6 Pro AP | 10 |
| 2 | Kitchen | U6 Pro AP | 10 |
| 3 | Camera location 1 | IP Camera | 30 |
| 4 | Camera location 2 | IP Camera | 30 |
| 5 | Living room | TV / Game system | 20 |

> **Note:** U6 Mesh Pro (neighbor) uses wireless uplink — no cable run required.

---

## 🔄 ISP Transition Plan

### Current (Verizon Router + UniFi Stack)
```
[Verizon Router] ──IP Passthrough──▶ [UDM Pro WAN] ──▶ [Network]
```
IP Passthrough eliminates double NAT. Verizon router acts as a transparent pass-through.

### After Fiber Cutover
```
[Fiber ONT] ──Ethernet──▶ [UDM Pro WAN] ──▶ [Network]
```
No UniFi reconfiguration required. Pull the Verizon router, plug ONT into UDM Pro WAN port. Done.

---

## 🏗️ Build Order

1. **Mount rack** in utility closet
2. **Pull all Cat6A runs** (cable before gear)
3. **Terminate wall plates** (keystone punch-downs at each drop)
4. **Install UDM Pro** — configure behind Verizon router via IP Passthrough
5. **Install USW-16-POE** — adopt into UDM Pro
6. **Install UPS 2U + PDU Pro** — power management
7. **Punch down patch panels** — terminate rack end of all runs
8. **Adopt APs** — U6 Pro x2 wired, U6 Mesh Pro wireless uplink
9. **Configure VLANs** — segment traffic per plan above
10. **Fiber cutover** — swap Verizon router for ONT direct connection

---

## 📐 Utility Closet Requirements

| Dimension | Minimum |
|-----------|---------|
| Width | 24" |
| Depth | 36–42" (including working clearance) |
| Wall space (rack footprint) | ~23.5"W × ~26"H |
| Floor space (if tower UPS) | 18"D × 7"W × 9"H |
| Working clearance in front | 18–24" |

**Ventilation:** NavePoint rack includes 2 built-in fans. Total rack heat load ~93W — low enough that a louvered door or small gap provides sufficient passive airflow. No dedicated HVAC required.

---

## 📚 CCNA Study Notes

Concepts practiced in this build:

- **VLANs & 802.1Q trunking** — traffic segmentation between networks
- **Inter-VLAN routing** — configured on UDM Pro
- **NAT & IP Passthrough** — eliminating double NAT behind ISP router
- **PoE (802.3af/at)** — powering APs and cameras over Cat6A
- **SFP+ / DAC cables** — 10G uplink between gateway and switch
- **Structured cabling standards** — Cat6A, TIA-568, punch-down termination
- **Wireless mesh** — wireless uplink topology for the U6 Mesh Pro
- **Firewall rules** — isolating neighbor and IoT VLANs

---

## 🔧 Tools Used

- Fish tape & cable pulling equipment
- Punch-down impact tool (110 blade)
- Cable toner / probe kit
- Label maker (for patch panel documentation)
- Voltage tester (before mounting rack)

---

## 📦 Recommended Suppliers

| Item | Source |
|------|--------|
| UniFi gear | [store.ui.com](https://store.ui.com) |
| Cat6A keystones & patch panels | [Monoprice](https://monoprice.com) / [Infinity Cable Products](https://infinitycableproducts.com) |
| Rack enclosure | [NavePoint on Amazon](https://amazon.com) |
| DAC cable | Amazon (10Gtek / Cablelist) |

---

## 📸 Build Progress

*Photos and progress updates will be added as the build progresses.*

---

## 📄 License

This project is documented for personal and educational use.  
Feel free to use it as a reference for your own home lab build.

---

*Built with 💙 alongside CCNA studies*
