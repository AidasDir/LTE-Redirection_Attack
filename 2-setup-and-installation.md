# Setup and Installation

This document details the process of setting up and installing the LTE-Redirection Attack system. It covers all necessary prerequisites, hardware requirements, and installation steps to establish a fully functioning environment for cellular network security research and testing. For information about actually executing the attack once installation is complete, see the "Running the Attack" documentation.

---

## Overview

The LTE-Redirection Attack system consists of three primary Docker containers that work together to create a controlled testing environment:
- **redirect_4_2g**: Fake LTE base station (OpenLTE-based)
- **osmo_egprs**: Complete 2G/EGPRS network (Osmocom-based)
- **asterisk**: VoIP PBX for call handling (Asterisk)

---

## Hardware Requirements

The system requires specific hardware to function properly:

| Component        | Requirement                                        | Notes                                               |
|------------------|----------------------------------------------------|-----------------------------------------------------|
| SDR for LTE      | BladeRF-xA4 (preferred), LimeSDR, USRP B200/ANTSDR | Used for the fake LTE base station                  |
| SDR for 2G/EGPRS | Same supported models as above                     | Must be a separate physical device                  |
| Computer         | Ubuntu 22.04                                       | Other Linux distributions may work but are untested |
| CPU              | Quad-core (minimum)                                | Higher performance recommended                      |
| RAM              | 8GB (minimum)                                      | 16GB+ recommended for smoother operation            |
| USB              | Multiple USB 3.0 ports                             | Sufficient bandwidth for SDR devices                |
| Internet         | Active connection                                  | For package downloads and traffic forwarding        |

---

## Prerequisites Check

Before installation, ensure you have:

1. **Root privileges** (all commands require sudo)
2. **SDR hardware** connected via USB
3. **Internet connection** for downloading dependencies
4. **Basic understanding of networking concepts**

---

## Installation Steps

### 1. Clone the Repository

```bash
git clone <repository-url>
cd LTE-Redirection_Attack
```

### 2. Run the Build Script

```bash
sudo ./build.sh
```

---

## Installation Details

The `build.sh` script performs the following key operations:

1. **Dependency Installation**:  
   Installs required packages: `chromium-browser`, `docker.io`, `docker-compose-v2`, `kconfig-frontends`, `docker-buildx`, `xterm`, `shellinabox`, `libbladerf2`, `tmux`
2. **Configuration Backup**:  
   Backs up any existing `.config` file to `.config.old`
3. **Network Preparation**:  
   Resets iptables rules with `reset_tables.sh`  
   Restarts Docker service  
   Refreshes network with DHCP
4. **Hardware Detection and Selection**:  
   Runs `scripts/check_hw.sh` which presents a menu-driven interface  
   User selects their SDR hardware type (BladeRF-xA4, LimeSDR, USRP B200/ANTSDR)  
   Generates a `.config` file used by all components
5. **Container Building**:  
   Builds Osmocom components with `osmocom.sh`  
   Builds Asterisk VoIP server with `asterisk.sh`  
   Builds OpenLTE redirector with `openlte.sh`

---

## Hardware Selection Process

The hardware selection is a critical part of the installation:
- The build script calls `scripts/check_hw.sh`, which presents a menu-driven interface.
- User selects their SDR hardware type and features.
- A `.config` file is generated and used by all components.

---

## Docker Container Architecture

After installation, the system creates three Docker containers that work together:
- `redirect_4_2g-example`
- `osmo_egprs-example`
- `asterisk-example`

---

## Verifying Installation

After running the build script, verify the installation:

### 1. Check Docker Containers

Run:
```bash
docker ps
```
Expected output should show three containers running:
- `redirect_4_2g-example`
- `osmo_egprs-example`
- `asterisk-example`

### 2. Verify SDR Hardware Detection
- For BladeRF: use `bladeRF-cli -p`
- For LimeSDR: use `LimeUtil --find`
- For USRP: use `uhd_find_devices`

### 3. Check Configuration Files

The installation creates multiple configuration files:
- `.config`: Main hardware configuration
- `osmo_egprs/configs/osmo-bsc.cfg`: BSC configuration
- `osmo_egprs/configs/osmo-sgsn.cfg`: SGSN configuration
- `osmo_egprs/configs/osmo-msc.cfg`: MSC configuration
- `osmo_egprs/configs/osmo-ggsn.cfg`: GGSN configuration

---

## Troubleshooting Installation Issues

| Issue                                       | Potential Solution                               |
|----------------------------------------------|--------------------------------------------------|
| "Script must be run as root"                | Use `sudo ./build.sh`                            |
| Docker errors                               | Check Docker status: `systemctl status docker`   |
|                                              | Restart Docker: `systemctl restart docker`       |
| SDR hardware not detected                   | Check physical connection                        |
|                                              | Verify with `lsusb`                              |
|                                              | Check permissions on `/dev/bus/usb`              |
| OpenLTE crash or libusb issues              | Ensure `udevadm trigger` completes before starting|
|                                              | Use different USB port for bandwidth issues      |

---

## Legal Warning

This system is designed exclusively for research, education, and authorized security testing. Unauthorized operation of fake base stations is illegal in most jurisdictions. Only use in controlled environments with proper authorization. Interfering with commercial networks may violate telecommunications laws.

---

## Next Steps

After successful installation, proceed to "Running the Attack" to learn how to configure and operate the system.

---

*Source: [DeepWiki LTE-Redirection_Attack Setup and Installation](https://deepwiki.com/AidasDir/LTE-Redirection_Attack/2-setup-and-installation)* 
