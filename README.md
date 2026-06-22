# Analog_Bridge
Core audio translation engine for the DVSwitch ecosystem.

## Installation Options

### Option 1: Debian Package Deployment (Recommended)
Go to the [Releases](https://github.com/DVSwitch/Analog_Bridge/releases) tab, select the latest stable release version, and download the `.deb` package matching your system architecture (`amd64`, `arm64`).
```bash
wget https://github.com/DVSwitch/Analog_Bridge/releases/download/v1.6.0/analog-bridge_1.6.0_amd64.deb
sudo apt install ./analog-bridge_1.6.0_amd64.deb
```

### Option 2: Cherry-Picking Specific Configuration Files
If you are updating or repairing an existing installation and only want a specific file without cloning the tree, use the Raw permalinks below:

* **Grab Latest Master Configuration Template (`Analog_Bridge.ini`):**
  ```bash
  wget https://raw.githubusercontent.com/DVSwitch/Analog_Bridge/main/config/Analog_Bridge.ini -O /etc/dvswitch/Analog_Bridge.ini
  ```

* **Grab Master Macro Template (`dvsm.macro`):**
  ```bash
  wget https://raw.githubusercontent.com/DVSwitch/Analog_Bridge/main/config/dvsm.macro -O /etc/dvswitch/dvsm.macro
  ```

* **Grab Latest Router/Control Script (`dvswitch.sh`):**
  ```bash
  wget https://raw.githubusercontent.com/DVSwitch/Analog_Bridge/main/scripts/dvswitch.sh -O /usr/local/bin/dvswitch.sh
  chmod +x /usr/local/bin/dvswitch.sh
  ```
