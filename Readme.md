# **PIMA Force – Hubitat Integration**

A complete integration of **PIMA Force** alarm panels with **Hubitat Elevation**, built around a reliable **Node-RED → Hubitat bridge**.  
This integration provides **real-time events**, accurate device representation, and full control of your PIMA Force alarm system directly from Hubitat.

---

## 🚀 What This Integration Provides

### **Real-Time Alarm System Visibility**
- **Instant zone events:** open, closed, tamper, alarm
- **All partition states:** ready, not ready, exit delay, entry delay, armed home/away, alarm
- **Panel-wide events:** troubles, heartbeat, connectivity
- **Sirens represented as true ON/OFF switch devices**

### **Direct Control from Hubitat**
- Arm Home / Arm Away
- Disarm
- Activate/deactivate sirens
- Read partition delays (exit/entry)
- Refresh panel configuration

### **Clean Hubitat Device Structure**

| Device Type                    | Purpose                                         |
|--------------------------------|-------------------------------------------------|
| PIMA Force Alarm Panel (parent)| Main controller and state aggregator            |
| Partition devices              | Arm/disarm + all partition state tracking       |
| Zone devices                   | Real-time open/close + alarm + tamper           |
| Siren devices                  | Pure ON/OFF output mapping                      |

> All child devices are created automatically upon receiving initial data from Node-RED.

### **Rock-Solid Communication Layer**

**Powered by Node-RED:**
- Persistent TCP session with the PIMA panel
- Auto-reconnect logic
- Full PIMA protocol decoding (`EVENT` / `OPERATION` / `DATA-REQ`)
- Heartbeat and bridge-state reporting
- Maker API communication to Hubitat
- Built-in debugging and test inject nodes

---

## 📝 Requirements

### **Before you begin**
- Installer password for your PIMA Force system.
- Panel must run **JSON-enabled firmware** (request update from PIMA support).

### **Hardware**
- PIMA Force alarm panel with LAN module
- Server running Node-RED (e.g., **Raspberry Pi**, **Proxmox VM**)

### **Software**
- **Hubitat Maker API** (built-in)
- **Node-RED 3.x or later**
- Required Node-RED palettes:
  - `node-red-contrib-buffer-parser`
  - `node-red-contrib-hubitat`
  - `node-red-dashboard` (optional)
  - `node-red-contrib-moment` (optional)
- **All drivers and flow files from this repository**

---

## 🛠️ Installation Guide

### 1. **PIMA Force Panel Configuration**
- Verify:
  - LAN module installed and enabled
  - Static IP address configured
  - Installer access available
  - JSON firmware installed
  - Node-RED server can reach panel over LAN

### 2. **Install Hubitat Drivers**
- Install 4 drivers from `/drivers/` folder:
  - **PIMA Force Alarm Panel** (*parent driver*)
  - **PIMA Force Partition**
  - **PIMA Force Zone**
  - **PIMA Force Siren**

> These are installed manually (not published through HPM yet).

### 3. **Create the Hubitat Parent Device**
- **Devices** → Add Virtual Device
- Name: e.g., `PIMA Force Alarm Panel`
- Select driver: `PIMA Force Alarm Panel`
- Save Device
- Child devices will appear automatically after initial sync.

### 4. **Configure Maker API**
- Create new Maker API instance
- Allow access to the parent device
- Copy:
  - **App ID**
  - **Access Token**
  - **Base URL**
- Paste in parent driver's preferences and Save

### 5. **Install the Node-RED Flow (Mandatory)**
- Import JSON flow from:  
  `/node-red/pima-flow.json`
- The flow includes:
  - Persistent TCP connection to the panel
  - Complete protocol parser
  - Automatic reconnect & health monitoring
  - Maker API update logic
  - Test inject nodes
  - Configuration sync (**Hubitat → Node-RED**)

#### **Required Node-RED Palettes:**  
Install before deploying:
- `node-red-contrib-buffer-parser`
- `node-red-contrib-hubitat`
- Optional: `node-red-dashboard`, `node-red-contrib-moment`

#### **Node-RED Configuration**
- **No manual entry** of Maker API URLs or PIMA credentials.
- Hubitat sends configuration via `syncConfig` command.
- **You must verify only:**
  - TCP node points to correct PIMA panel IP & port
  - Hubitat nodes use installed palette

> Deploy the flow. Node-RED will connect to your panel and start sending events to Hubitat.

### 6. **Initial Sync**
- When the PIMA panel responds to `DATA-REQ`:
  - All partitions are created
  - All zones created with correct labels
  - All sirens/outputs created
  - Panel status becomes active

> You will now see live events in Hubitat.

---

## 🧑‍💻 Using the Integration

### **Arm / Disarm**
- Partition device commands:
  - `armHome`
  - `armAway`
  - `disarm`

### **Zone Activity**
- Each zone device reports:
  - `open` / `closed`
  - Alarm events
  - Tamper
  - `lastEventTs`

### **Siren Control**
- Siren devices: simple ON/OFF switch, matching PIMA behavior.

### **Connectivity**
- Parent device reports:
  - `bridgeState`
  - `heartbeatTs`
- If unreachable, presence switches to `not present`.

---

## 🔧 What You Must Configure

### **In Hubitat**
1. Install 4 drivers
2. Create parent device
3. Set Maker API parameters in device preferences
4. Save

### **In Node-RED**
1. Verify TCP node uses correct PIMA LAN IP/port
2. Ensure `node-red-contrib-hubitat` is installed
3. Deploy the flow

### **In Your LAN**
- Ensure PIMA LAN module has a static IP
- Ensure Node-RED can reach both Hubitat and PIMA

---

## 🌐 How It Works

```
[PIMA Panel] ← TCP → [Node-RED Parser] → Maker API → [Hubitat Parent Device]
                                                      ↓ auto-creates
                                            [Partitions] [Zones] [Sirens]
```

---

## 🔄 Updates & Maintenance

### **Updating the Node-RED Flow**
- Import updated JSON, or adjust individual nodes.

### **Updating Hubitat Drivers**
- Paste updated code → Save → Done.
- Child devices remain intact.

### **PIMA Firmware**
- Some systems require manual firmware update from PIMA support.

---

## 🆘 Troubleshooting

### **No child devices appear**
- Maker API misconfigured
- Node-RED flow not deployed
- TCP connection to PIMA failing
- Try sending `syncConfig` from Hubitat

### **Wrong timestamps**
- Ensure Node-RED system time is correct
- Check Hubitat’s location/timezone settings

### **Slow event reporting**
- PIMA panels send events with a built-in ~1 second minimum interval.

---

## 💬 Support

For bugs or feature requests, **open a GitHub Issue in this repository**.
