# LTE-Redirection Attack - System Overview

## Overview

The LTE-Redirection_Attack repository is a comprehensive research platform designed for cellular network security testing and analysis. This system enables the creation of a controlled environment for testing mobile device vulnerabilities through network downgrade attacks, specifically by forcing LTE-connected devices to downgrade to less secure 2G/EGPRS networks.

## System Purpose and Capabilities

The LTE-Redirection Attack platform enables researchers and security professionals to:

1. Deploy a fake LTE base station (eNodeB) that mobile devices will connect to
2. Force connected devices to downgrade from secure LTE connections to less secure 2G/EGPRS
3. Capture and analyze traffic through a controlled 2G network environment
4. Intercept and analyze voice calls using an integrated Asterisk PBX
5. Configure the attack for different mobile network operators (MNOs)

The system is containerized using Docker for consistent deployment and includes support for various Software Defined Radio (SDR) hardware options.

---

## System Architecture

The LTE-Redirection Attack platform consists of three primary Docker containers that work together to execute the attack:

1. **redirect_4_2g Container**: Implements a fake LTE base station using modified OpenLTE that broadcasts selected Mobile Country Code (MCC) and Mobile Network Code (MNC) to target specific networks and configures redirection parameters to force devices to downgrade to 2G.
2. **osmo_egprs Container**: Implements a complete Osmocom-based 2G network stack including HLR, MSC, BSC, SGSN, GGSN components for full cellular functionality, providing data services through GTP tunneling.
3. **asterisk Container**: Implements Asterisk PBX for call handling and integration with the Osmocom voice stack.
4. **Network Routing**: Configures NAT and IP forwarding to route traffic between captured devices and the internet.

---

## Hardware Requirements

The system requires at least two Software Defined Radios (SDRs):

1. **Primary SDR for LTE Base Station:**  
   Handles the LTE redirection attack.  
   Compatible hardware: BladeRF-xA4 (preferred), LimeSDR, USRP B200/ANTSDR
2. **Secondary SDR for 2G/EGPRS Base Station:**  
   Handles the 2G network that devices connect to after redirection.  
   Compatible hardware: Same options as above, but requires a separate physical device

Additional hardware requirements include:
- Computer running Ubuntu 22.04 (tested platform)
- Sufficient USB ports and bandwidth for SDR devices
- Minimum 8GB RAM and quad-core CPU recommended
- Internet connection for package downloads and traffic forwarding

---

## Core Components in Detail

### LTE Redirector (redirect_4_2g)

The LTE redirector component is a modified version of OpenLTE that creates a fake LTE base station. Its primary functions are:

1. Broadcasting a selected operator's MCC/MNC to attract target devices
2. Configuring specific LTE bands and frequencies (EARFCN values)
3. Forcing connected devices to downgrade to 2G through RRC Connection Release messages
4. Providing a telnet interface (port 30000) for runtime configuration

Key configuration parameters include:
- MCC (Mobile Country Code)
- MNC (Mobile Network Code)
- Tracking Area Code (TAC)
- LTE Band
- Downlink EARFCN (frequency)
- Transmit and receive gain settings

### 2G/EGPRS Network (osmo_egprs)

The 2G/EGPRS network uses the Osmocom stack to provide a complete cellular network that devices connect to after being redirected. Components include:

- **osmo-hlr**: Home Location Register, stores subscriber information
- **osmo-msc**: Mobile Switching Center, handles call routing
- **osmo-bsc**: Base Station Controller, manages BTS
- **osmo-sgsn**: Serving GPRS Support Node, handles data sessions
- **osmo-ggsn**: Gateway GPRS Support Node, provides internet access
- **osmo-trx**: Transceiver interface to SDR hardware
- **osmo-bts**: Base Transceiver Station, the radio interface
- **osmo-pcu**: Packet Control Unit, handles GPRS/EDGE data

Key configuration files are located in `osmo_egprs/configs/` and include:
- `osmo-bsc.cfg`: BSC configuration, including remote IP address
- `osmo-sgsn.cfg`: SGSN configuration, including listen IP address
- `osmo-msc.cfg`: MSC configuration
- `osmo-ggsn.cfg`: GGSN configuration

### Asterisk VoIP Integration

The Asterisk VoIP server is used to handle voice calls in the captured network. It connects to Osmocom through the osmo-sip-connector. Key configuration files in the `asterisk/` directory include:
- `sip.conf`: SIP protocol configuration
- `extensions.conf`: Call routing configuration

The Asterisk container starts with high verbosity for debugging purposes.

---

## Attack Flow Sequence

1. Deploy the LTE redirector (redirect_4_2g) to attract and connect target devices.
2. Force connected devices to downgrade to 2G using RRC Connection Release messages.
3. Devices connect to the 2G/EGPRS network (osmo_egprs), which provides full cellular and data services.
4. Voice calls are handled and can be intercepted/analyzed via the Asterisk PBX.
5. All network traffic is routed through NAT and IP forwarding for analysis or internet access.

---

## Setup and Operation Scripts

### build.sh
Prepares the environment, installs dependencies, configures hardware, and builds Docker containers:
1. Installs required dependencies
2. Backs up any existing configuration
3. Resets iptables rules
4. Restarts Docker service
5. Runs hardware detection and selection
6. Builds containers for selected hardware and components

### run.sh
Launches the attack environment, configures network interfaces, and opens interactive terminals:
1. Prompts for forwarding interface device and IP
2. Configures interface masquerading with `srsepc_if_masq.sh`
3. Updates Osmocom configs with the provided IP
4. Runs `choose_interface.sh` for operator selection
5. Starts all Docker containers
6. Opens terminals for 2G, LTE redirection, and Asterisk
7. Sets up forwarding and NAT rules
8. Provides telnet access to the containers

---

## Operator Configuration

The system can be configured to target specific mobile network operators by setting the appropriate MCC and MNC values. The `choose_interface.sh` script provides a menu-driven selection for common operators, and the selection is stored in an `operator` file used by `scripts/redir.sh`.

Operator-specific configuration is handled through Python scripts:
- `scripts/telnet_orange.py`: Configuration for Orange (MCC: 208, MNC: 01)
- `scripts/telnet_sfr.py`: Configuration for SFR (MCC: 208, MNC: 10)
- `scripts/telnet_free.py`: Configuration for Free (MCC: 208, MNC: 15)
- `scripts/telnet_bouygues.py`: Configuration for Bouygues/Lyca (MCC: 208, MNC: 20/25)

These scripts automate the configuration of the LTE redirector for each operator's specific network parameters.

---

## Legal Considerations

This tool is intended for security research, education, and authorized penetration testing to demonstrate vulnerabilities in cellular networks. Running fake base stations is illegal in many jurisdictions without proper authorization and may violate telecommunications laws.

The platform showcases how malicious actors could force devices to connect to less secure networks, potentially exposing them to interception or manipulation. This knowledge is essential for developing countermeasures and improving network security.

---

*Source: [DeepWiki LTE-Redirection_Attack Overview](https://deepwiki.com/AidasDir/LTE-Redirection_Attack/1-overview)*

