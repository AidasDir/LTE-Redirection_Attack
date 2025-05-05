# LTE-Redirection Attack - Technical Documentation

## Table of Contents

1. [Overview and Architecture](#overview-and-architecture)
   - [Purpose of the Application](#purpose-of-the-application)
   - [System Architecture](#system-architecture)
   - [Hardware Requirements](#hardware-requirements)

2. [Complete File Structure](#complete-file-structure)
   - [Top-Level Files](#top-level-files)
   - [Main Directories](#main-directories)
   - [Configuration Files](#configuration-files)
   - [Script Files](#script-files)

3. [Step-by-Step Installation Guide](#step-by-step-installation-guide)
   - [Prerequisites](#prerequisites)
   - [Installation Process (build.sh)](#installation-process-buildsh)
   - [Hardware Detection and Selection](#hardware-detection-and-selection)

4. [Detailed Usage Instructions](#detailed-usage-instructions)
   - [Running the Application (run.sh)](#running-the-application-runsh)
   - [2G EGPRS Base Station Configuration](#2g-egprs-base-station-configuration)
   - [LTE Redirection Configuration](#lte-redirection-configuration)
   - [Network Forwarding and NAT](#network-forwarding-and-nat)
   - [Operator-Specific Configuration](#operator-specific-configuration)

5. [Technical Deep Dive](#technical-deep-dive)
   - [Osmocom Components](#osmocom-components)
   - [OpenLTE Redirection](#openlte-redirection)
   - [Asterisk VoIP Integration](#asterisk-voip-integration)
   - [SDR Hardware Integration](#sdr-hardware-integration)

6. [Troubleshooting Guide](#troubleshooting-guide)
   - [Common Issues](#common-issues)
   - [Log Files](#log-files)
   - [Debug Commands](#debug-commands)

7. [Legal and Security Considerations](#legal-and-security-considerations)

---

## Overview and Architecture

### Purpose of the Application

The LTE-Redirection Attack is a research and testing platform for cellular network security. It creates a multi-component environment that:

1. Deploys a fake LTE base station to initially capture mobile devices
2. Forces devices to downgrade from secure LTE connections to less secure 2G/EGPRS
3. Establishes a controlled 2G EGPRS network that the redirected devices connect to
4. Provides VoIP functionality through Asterisk for call interception or manipulation
5. Manages network routing to allow captured devices to access the internet or other network resources

This platform is designed for security research, education, and authorized penetration testing to demonstrate vulnerabilities in cellular networks. It showcases how a malicious actor could force devices to connect to less secure networks, potentially exposing them to interception or manipulation.

### System Architecture

The application consists of several interconnected components, each running in Docker containers:

1. **LTE Component (redirect_4_2g):**
   - Implements a fake LTE base station (eNodeB) using modified OpenLTE
   - Broadcasts selected MCC/MNC to target specific networks
   - Configures redirection parameters to force devices to downgrade to 2G

2. **2G/EGPRS Component (osmo_egprs):**
   - Implements a complete Osmocom-based 2G network stack
   - Includes HLR, MSC, BSC, SGSN, GGSN for complete cellular functionality
   - Provides data services through GTP tunneling

3. **VoIP Component (asterisk):**
   - Implements Asterisk PBX for call handling
   - Integrates with the Osmocom voice stack

4. **Builder Component (builder):**
   - Manages SDR hardware selection and configuration
   - Builds necessary drivers and interfaces

5. **Network Routing:**
   - Configures NAT and IP forwarding to route traffic between captured devices and internet

### Hardware Requirements

The system requires at least two Software Defined Radios (SDRs):

1. **Primary SDR for LTE Base Station:**
   - Handles the LTE redirection attack
   - Compatible hardware: BladeRF-xA4 (preferred), LimeSDR, USRP B200/ANTSDR

2. **Secondary SDR for 2G/EGPRS Base Station:**
   - Handles the 2G network that devices connect to after redirection
   - Compatible hardware: Same as above, but requires a separate physical device

Additional hardware requirements:
- Computer running Ubuntu 22.04 (tested)
- Sufficient USB ports and bandwidth for SDR devices
- Minimum 8GB RAM and quad-core CPU recommended
- Internet connection for package downloads and potential traffic forwarding

---

## Complete File Structure

### Top-Level Files

- **`build.sh`**: Main build script that:
  - Installs dependencies (chromium-browser, docker, kconfig-frontends, etc.)
  - Backs up and manages configuration
  - Resets iptables
  - Manages Docker services
  - Runs hardware selection through `scripts/check_hw.sh`
  - Builds and starts containers for Osmocom, Asterisk, and OpenLTE

- **`run.sh`**: Main execution script that:
  - Checks for root privileges
  - Gets interface information from user
  - Configures Osmocom base station IPs
  - Manages Docker containers
  - Handles udev and device permissions
  - Loads kernel modules
  - Launches terminals for different components
  - Sets up NAT and routing

- **`reset_tables.sh`**: Cleans iptables rules and chains for a fresh network configuration

- **`srsepc_if_masq.sh`**: Configures IP forwarding and NAT for the specified interface

- **`osmocom.sh`**: Builds and starts Osmocom components in Docker containers

- **`asterisk.sh`**: Builds and starts the Asterisk VoIP server

- **`openlte.sh`**: Builds and starts the OpenLTE redirector

- **`choose_interface.sh`**: Runs menu-driven interface selection and propagates the selection to all components

- **`forward.sh`**: Sets up IP forwarding rules for interfaces

### Main Directories

- **`scripts/`**: Contains utility scripts for running components
  - `2G.sh`: Launches the 2G EGPRS Docker container
  - `asterisk.sh`: Launches the Asterisk Docker container
  - `check_hw.sh`: Script for SDR hardware detection
  - `redir.sh`: Launches the LTE redirector and operator-specific telnet scripts
  - `telnet_*.py`: Operator-specific scripts for configuring the LTE redirector

- **`redirect_4_2g/`**: Contains the LTE redirection components
  - `configs/`: LTE redirector configuration files
  - `scripts/`: Scripts for the LTE redirector
  - `Dockerfile`: Builds the LTE redirector container

- **`asterisk/`**: Contains the Asterisk VoIP server configuration
  - `extensions.conf`: Call routing configuration
  - `sip.conf`: SIP protocol configuration
  - `Dockerfile`: Builds the Asterisk container

- **`osmo_egprs/`**: Contains the 2G/EGPRS base station components
  - `configs/`: Configuration files for all Osmocom components
  - `scripts/`: Scripts for the 2G/EGPRS base station
  - `Dockerfile`: Builds the 2G/EGPRS container

- **`builder/`**: Contains hardware-specific builders
  - `selected_stuff`: Menu configuration for SDR hardware selection
  - Subdirectories for different SDR types (bladerf, limesdr, usrp)

- **`osmocom_installation/`**: Contains installation scripts for all Osmocom components
  - Subdirectories for each component (hlr, msc, bsc, etc.)
  - `osmo-all.sh`: Script to start all Osmocom components

- **`openlte_redir_install/`**: Contains the OpenLTE redirector installation
  - `Dockerfile`: Builds the OpenLTE redirector container
  - `uhd.patch`: Patches for the UHD driver

### Configuration Files

Key configuration files that may need modification:

- **`osmo_egprs/configs/osmo-bsc.cfg`**: BSC configuration, including remote IP address

- **`osmo_egprs/configs/osmo-sgsn.cfg`**: SGSN configuration, including listen IP address

- **`osmo_egprs/configs/osmo-msc.cfg`**: MSC configuration

- **`osmo_egprs/configs/osmo-ggsn.cfg`**: GGSN configuration

- **`.config`**: Generated by hardware selection, propagated to all components

### Script Files

Key script files for running components:

- **`scripts/2G.sh`**: Runs the 2G base station with all necessary privileges

- **`scripts/redir.sh`**: Runs the LTE redirector, sets up Python environment, and runs operator-specific telnet scripts

- **`scripts/telnet_*.py`**: Operator-specific scripts (Orange, SFR, Free, Bouygues) that automate the configuration of the LTE redirector

---

## Step-by-Step Installation Guide

### Prerequisites

1. **Operating System:**
   - Ubuntu 22.04 (tested platform)
   - Running with root privileges

2. **Hardware:**
   - At least two compatible SDR devices connected via USB
   - Supported hardware: BladeRF-xA4, LimeSDR, USRP B200/ANTSDR

3. **Knowledge Requirements:**
   - Basic understanding of networking and Linux commands
   - Familiarity with cellular network concepts (MCC, MNC, EARFCN, etc.)

### Installation Process (build.sh)

1. **Clone the Repository:**
   ```bash
   git clone <repository-url>
   cd LTE-Redirection_Attack
   ```

2. **Run the Build Script:**
   ```bash
   sudo ./build.sh
   ```

3. **What the Build Script Does:**
   - Installs required dependencies:
     ```
     chromium-browser docker.io docker-compose-v2 kconfig-frontends docker-buildx xterm shellinabox libbladerf2 tmux
     ```
   - Backs up any existing `.config` file to `.config.old`
   - Resets iptables rules with `reset_tables.sh`
   - Restarts Docker service
   - Refreshes network with DHCP
   - Runs hardware detection and selection with `scripts/check_hw.sh`
   - Conditionally builds hardware-specific containers
   - Builds and starts Osmocom components with `osmocom.sh`
   - Builds and starts Asterisk with `asterisk.sh`
   - Builds and starts OpenLTE redirector with `openlte.sh`

### Hardware Detection and Selection

1. **Running Hardware Selection:**
   - The build script calls `scripts/check_hw.sh`
   - This runs a menu-driven interface with `kconfig-mconf builder/selected_stuff`

2. **Selecting Your Hardware:**
   - Navigate the menu using arrow keys
   - Press 'Y' to select an option, 'N' to deselect
   - Select your SDR hardware type (BladeRF-xA4, LimeSDR, USRP B200/ANTSDR)
   - Select additional features (OpenLTE redirection, Asterisk, Osmocom network)

3. **Configuration Generation:**
   - After selection, a `.config` file is generated
   - This file is used by all components to determine which hardware to use

---

## Detailed Usage Instructions

### Running the Application (run.sh)

1. **Start the Application:**
   ```bash
   sudo ./run.sh
   ```

2. **Input Required When Prompted:**
   - **Forwarding Interface Device**: Enter the network interface for internet access (e.g., eth0, wlan0)
   - **Forwarding Interface IP**: Enter the IP address of the interface
   - **Restart Docker**: Confirm whether to restart the Docker service (default: Y)

3. **Automatic Setup Process:**
   - Runs `srsran_performance` script
   - Configures interface masquerading with `srsepc_if_masq.sh`
   - Updates Osmocom configs with the provided IP
   - Triggers udev for device detection
   - Runs `choose_interface.sh` for operator selection
   - Loads the GTP kernel module
   - Stops udev services to prevent conflicts
   - Starts all Docker containers:
     - `osmo_egprs/docker-compose up -d`
     - `redirect_4_2g/docker-compose up -d`
     - `asterisk/docker-compose up -d`
   - Opens terminals for 2G, LTE redirection, and Asterisk
   - Sets DNS to Google (8.8.8.8)
   - Sets up forwarding and NAT
   - Opens a telnet connection to the container

### 2G EGPRS Base Station Configuration

1. **Automatic Terminal Launch:**
   - A terminal for the 2G base station opens automatically with `scripts/2G.sh`
   - This runs the Osmocom container with necessary privileges

2. **Manual Setup in the 2G Terminal:**
   - Run the following commands in the opened terminal:
     ```bash
     ./tun.sh
     ./osmo-all.sh start
     osmo-trx-uhd
     ```

3. **Configuration Files:**
   - Key configuration is in `osmo_egprs/configs/`
   - Critical files that may need adjustment:
     - `osmo-bsc.cfg`: Contains remote IP settings (automatically updated by run.sh)
     - `osmo-sgsn.cfg`: Contains listen IP settings (automatically updated by run.sh)
     - Make manual edits if automatic updates don't work correctly

### LTE Redirection Configuration

1. **Automatic Terminal Launch:**
   - A terminal for LTE redirection opens automatically with `scripts/redir.sh`
   - This runs the OpenLTE container with necessary privileges and starts the appropriate telnet script

2. **Operator-Specific Configuration:**
   - Based on operator selection in `choose_interface.sh`, one of the following scripts runs:
     - `telnet_orange.py`: Configuration for Orange
     - `telnet_sfr.py`: Configuration for SFR
     - `telnet_free.py`: Configuration for Free
     - `telnet_bouygues.py`: Configuration for Bouygues/Lyca

3. **Manual Configuration (if needed):**
   - If automatic configuration fails, connect to the LTE redirector:
     ```bash
     telnet 172.17.0.2 30000
     ```
   - Enter the following commands:
     ```
     write mcc <mcc>            # Mobile Country Code for target network
     write mnc <mnc>            # Mobile Network Code for target network
     write tracking_area_code <tac>  # Tracking Area Code
     write band <band>          # LTE band to use (e.g., 3 for Band 3)
     write dl_earfcn <earfcn>   # Downlink EARFCN
     write tx_gain 80           # Transmit gain (adjust as needed)
     write rx_gain 30           # Receive gain (adjust as needed)
     start                      # Start the LTE redirector
     ```

### Network Forwarding and NAT

1. **Automatic Configuration:**
   - `run.sh` automatically sets up NAT and forwarding with:
     - `forward.sh`: Sets up IP forwarding and interface masquerading
     - `srsepc_if_masq.sh`: Configures masquerading for the specified interface

2. **Manual Configuration (if needed):**
   - If automatic configuration fails, run:
     ```bash
     bash reset_tables.sh
     ./srsepc_if_masq.sh <your_interface>
     ```
   - Add additional NAT rules if needed:
     ```bash
     iptables -A POSTROUTING -t nat -s 176.16.32.0/20 ! -d 176.32.16.0/20 -j MASQUERADE
     ```

### Operator-Specific Configuration

1. **Operator Selection:**
   - Run `choose_interface.sh` to select the operator
   - This creates an `operator` file that is used by `scripts/redir.sh`

2. **Available Operators:**
   - **Orange**: French operator (MCC: 208, MNC: 01)
   - **SFR**: French operator (MCC: 208, MNC: 10)
   - **Free**: French operator (MCC: 208, MNC: 15)
   - **Bouygues/Lyca**: French operator (MCC: 208, MNC: 20/25)

3. **Customizing for Other Operators:**
   - Create a new `telnet_*.py` script based on existing ones
   - Modify the MCC, MNC, band, and EARFCN values for your target operator
   - Update `scripts/redir.sh` to include your new operator

---

## Technical Deep Dive

### Osmocom Components

1. **Core Components:**
   - **libosmocore**: Core libraries for Osmocom
   - **osmo-hlr**: Home Location Register, stores subscriber information
   - **osmo-msc**: Mobile Switching Center, handles call routing
   - **osmo-mgw**: Media Gateway, handles voice traffic
   - **osmo-stp**: Signaling Transfer Point, routes SS7 messages
   - **osmo-ggsn**: Gateway GPRS Support Node, provides internet access
   - **osmo-sgsn**: Serving GPRS Support Node, handles data sessions
   - **osmo-bsc**: Base Station Controller, manages BTS
   - **osmo-sip**: SIP connector for VoIP integration
   - **osmo-trx**: Transceiver interface to SDR hardware
   - **osmo-bts**: Base Transceiver Station, the radio interface
   - **osmo-pcu**: Packet Control Unit, handles GPRS/EDGE data

2. **Configuration Files:**
   - Located in `osmo_egprs/configs/`
   - These files control the behavior of each component
   - Key files to modify for custom deployments:
     - `osmo-bsc.cfg`: BSC configuration
     - `osmo-sgsn.cfg`: SGSN configuration
     - `osmo-msc.cfg`: MSC configuration
     - `osmo-ggsn.cfg`: GGSN configuration

3. **Running Osmocom:**
   - Started by `scripts/2G.sh`
   - Within the container, run:
     ```bash
     ./tun.sh              # Sets up TUN interface for data
     ./osmo-all.sh start   # Starts all Osmocom components
     osmo-trx-uhd          # Starts the transceiver with UHD driver
     ```

### OpenLTE Redirection

1. **Core Components:**
   - **LTE_fdd_enodeb**: Modified OpenLTE eNodeB implementation
   - **telnet interface**: Control interface on port 30000

2. **Configuration Parameters:**
   - **MCC**: Mobile Country Code (e.g., 208 for France)
   - **MNC**: Mobile Network Code (e.g., 01 for Orange)
   - **tracking_area_code**: TAC for the emulated network
   - **band**: LTE band to use (e.g., 3, 7, 20)
   - **dl_earfcn**: Downlink EARFCN value determining frequency
   - **tx_gain/rx_gain**: Transmit and receive gain settings

3. **Telnet Scripts:**
   - Located in `scripts/`
   - `telnet_orange.py`, `telnet_sfr.py`, etc.
   - These scripts automate the configuration of the LTE redirector
   - They set random cell IDs and operator-specific parameters
   - Parameters include MCC, MNC, band, EARFCN, and gain settings

### Asterisk VoIP Integration

1. **Core Components:**
   - **Asterisk PBX**: Open-source telephony engine
   - **SIP integration**: Connects to Osmocom through osmo-sip-connector

2. **Configuration Files:**
   - Located in `asterisk/`
   - `sip.conf`: SIP protocol configuration
   - `extensions.conf`: Call routing configuration

3. **Running Asterisk:**
   - Started by `scripts/asterisk.sh`
   - Runs with high verbosity for debugging (-cvvvvvvvvvv)

### SDR Hardware Integration

1. **Supported Hardware:**
   - **BladeRF-xA4**: Preferred for this application
   - **LimeSDR**: Alternative SDR
   - **USRP B200/ANTSDR**: Alternative SDR

2. **Hardware Selection:**
   - Configured in `builder/selected_stuff`
   - Menu-driven selection through `kconfig-mconf`
   - Generates `.config` file used by all components

3. **Hardware-Specific Configuration:**
   - `builder/bladerf/`: BladeRF-specific configuration
   - `builder/limesdr/`: LimeSDR-specific configuration
   - `builder/usrp/`: USRP-specific configuration

---

## Troubleshooting Guide

### Common Issues

1. **Script must be run as root:**
   - Both `build.sh` and `run.sh` require root privileges
   - Always use `sudo ./build.sh` and `sudo ./run.sh`

2. **Docker or network errors:**
   - Ensure Docker is installed and running
   - Check Docker status with `systemctl status docker`
   - Restart Docker with `systemctl restart docker`

3. **SDR hardware not detected:**
   - Ensure SDR hardware is connected before running
   - Verify with `lsusb`
   - Check permission issues with `/dev/bus/usb`

4. **LTE redirector not starting:**
   - Check that the correct MCC/MNC is set
   - Verify EARFCN is appropriate for the selected band
   - Adjust tx_gain/rx_gain values
   - Check SDR hardware connection

5. **2G base station not starting:**
   - Verify the correct interface IPs in configurations
   - Check that all Osmocom components start correctly
   - Ensure SDR hardware is properly connected

6. **Network forwarding/NAT issues:**
   - Verify IP forwarding is enabled with `sysctl net.ipv4.ip_forward`
   - Check iptables rules with `iptables -t nat -L`
   - Reset iptables with `reset_tables.sh`

7. **OpenLTE crash or libusb issues:**
   - Make sure `udevadm trigger` completes before starting
   - Use a different port if USB bandwidth is insufficient

### Log Files

1. **Docker Logs:**
   - Check logs with `docker logs <container-name>`
   - For specific containers:
     - `docker logs osmo_egprs-example`
     - `docker logs redirect_4_2g-example`
     - `docker logs asterisk-example`

2. **Component Logs:**
   - Osmocom components log to the terminal
   - Asterisk logs to the terminal with high verbosity
   - OpenLTE logs to the terminal and telnet session

### Debug Commands

1. **Check Docker Containers:**
   ```bash
   docker ps
   ```

2. **Check Network Interfaces:**
   ```bash
   ip addr
   ```

3. **Check Routing:**
   ```bash
   ip route
   ```

4. **Check NAT Rules:**
   ```bash
   iptables -t nat -L
   ```

5. **Check SDR Hardware:**
   ```bash
   lsusb
   uhd_find_devices  # For USRP
   bladeRF-cli -p    # For BladeRF
   LimeUtil --find   # For LimeSDR
   ```

---

## Legal and Security Considerations

1. **Legal Warning:**
   - This tool is for research, education, and authorized security testing only
   - Unauthorized operation of fake base stations is illegal in most jurisdictions
   - Use only in controlled environments with proper authorization
   - Interfering with commercial networks may violate telecommunications laws

2. **Security Implications:**
   - This tool demonstrates vulnerabilities in cellular networks
   - It shows how devices can be forced to downgrade from secure to insecure networks
   - It enables potential interception of voice and data traffic

3. **Responsible Disclosure:**
   - If security vulnerabilities are discovered, follow responsible disclosure
   - Contact affected vendors, operators, or security researchers
   - Allow time for fixes before public disclosure

4. **Documentation Purpose:**
   - This documentation is provided for educational purposes
   - Understanding security vulnerabilities helps improve network security
   - Knowledge of attack vectors is essential for developing countermeasures 
