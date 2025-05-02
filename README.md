# LTE-Redirection_Attack

## Overview

This project is a research and testing platform for cellular network redirection attacks, specifically targeting LTE and 2G (EGPRS) networks. It automates the setup of a multi-component environment using Docker containers, SDR (Software Defined Radio) hardware, and open-source telecom stacks (Osmocom, OpenLTE, Asterisk). The system is designed to:

- Deploy fake base stations (BTS) for 2G and LTE.
- Redirect mobile devices to insecure or controlled networks.
- Provide a VoIP core (Asterisk) for call handling.
- Support multiple SDR hardware types (BladeRF, LimeSDR, USRP, etc.).
- Automate network, firewall, and hardware configuration for rapid testing.

**Intended Use:**  
This platform is for research, education, and lawful security testing only. It requires root privileges and direct hardware access.

---

## 1. Installation

1. **Clone the repository and enter the project directory:**
   ```bash
   git clone <repo_url>
   cd LTE-Redirection_Attack
   ```
2. **Run the build script:**
   ```bash
   sudo ./build.sh
   ```
   - Follow the hardware selection menu to choose your SDR and features (Osmocom, OpenLTE, Asterisk).
   - The script will install dependencies, reset firewall rules, and build all required Docker containers.
   - No manual intervention should be needed if the script completes successfully.

---

## 2. Running the LTE-Redirection_Attack

1. **Run the main script:**
   ```bash
   sudo ./run.sh
   ```
   - Respond to prompts for network interface and IP address.
   - The script will configure NAT, start all containers, and open interactive terminals for 2G, LTE redirection, and Asterisk.

2. **Monitor the terminals:**
   - Terminals for 2G, LTE redirection, and Asterisk will open automatically.
   - The system is now ready for attack simulation or research.

---

## 3. Troubleshooting & Manual Steps (Fallback)

If the automation does not complete all steps, use the following manual interventions:

### a. Edit Osmocom Config Files
- If the automation does not set the correct IPs in `osmo-bsc.cfg` and `osmo-sgsn.cfg`, edit them manually:
  - In `osmo_egprs/configs/osmo-bsc.cfg`, set:
    ```
    0 remote ip <your_internet_interface_ip>
    ```
  - In `osmo_egprs/configs/osmo-sgsn.cfg`, set:
    ```
    listen <your_internet_interface_ip>
    ```

### b. Start Osmocom Services Manually
- If the Osmocom terminal does not start the required services, run:
  ```bash
  ./tun.sh
  ./osmo-all.sh start
  osmo-trx-uhd
  ```

### c. Network Forwarding/NAT
- If network forwarding is not working, run:
  ```bash
  bash reset_iptables.sh
  ./srsepc_if_masq.sh <your_interface>
  # If needed:
  iptables -A POSTROUTING -t nat -s 176.16.32.0/20 ! -d 176.32.16.0/20 -j MASQUERADE
  ```

### d. LTE Node Redirector Configuration
- If the LTE redirector is not configured automatically, use the provided Python scripts in `scripts/` (e.g., `telnet_orange.py`) or connect via telnet and enter the following commands:
  ```
  write mcc <replace>
  write mnc <replace>
  write tracking_area_code <replace>
  write band <replace>
  write dl_earfcn <replace>
  write tx_gain 80
  write rx_gain 30
  start
  ```

---

## 4. Additional Notes

- **SDR Hardware:** You need at least two SDRs (one for LTE, one for 2G) for a full real-time redirection attack.
- **Root Privileges:** All scripts must be run as root (sudo).
- **Logs:** Use `docker logs <container_name>` or check the running terminals for service logs.
- **Stopping the Environment:** Use `docker compose down` in each service directory or reboot to reset network/firewall changes.
- **Security Warning:** Running fake base stations is illegal in many jurisdictions without proper authorization.

---

## 5. References
- See the original `README_old.md` for more context and manual steps.
- For video walkthrough: [build](https://youtu.be/0aruLybY__w) 
