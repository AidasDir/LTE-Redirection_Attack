# Project Documentation

## Project Index

### Top-Level Files and Directories
- `build.sh`: Main build/setup script for installing dependencies and preparing the environment.
- `run.sh`: Main script to launch the system, configure interfaces, and start services/containers.
- `reset_tables.sh`: Script for resetting database or system tables.
- `srsran_performance`: Likely a script or binary for SRSRAN performance testing.
- `run_menu`: Possibly a menu-driven script for running options.
- `srsepc_if_masq.sh`: Script for interface masquerading (NAT) for SRS EPC.
- `osmocom.sh`: Script to set up or start Osmocom components.
- `choose_interface.sh`: Script to select a network interface.
- `dns_forward.sh`: Script for DNS forwarding.
- `docker-compose.yml`: Top-level Docker Compose file (if used).
- `forward.sh`: Script for forwarding network traffic.
- `init_pyenv.sh`: Script to initialize a Python environment.
- `openlte.sh`: Script to set up or start OpenLTE.
- `operator`: Likely a config or data file.
- `.config`, `.config.old`: Configuration files.
- `Dockerfile`: Top-level Dockerfile (if used).
- `LICENSE`: License file.
- `README.md`: Existing readme file.

#### Directories
- `scripts/`: Contains utility and helper scripts (e.g., 2G.sh, asterisk.sh, check_hw.sh, telnet scripts).
- `builder/`: Contains Dockerfiles, compose files, and subdirectories for different SDR hardware (e.g., bladerf, limesdr, usrp, ubuntu, end_hw).
- `osmo_egprs/`: Contains configs, scripts, Dockerfiles, and binaries for Osmocom EGPRS.
- `redirect_4_2g/`: Contains configs, scripts, Dockerfiles for 2G redirection.
- `asterisk/`: Contains configs, Dockerfiles, and compose files for Asterisk PBX.
- `openlte_redir_install/`: Contains Dockerfiles, compose files, and patches for OpenLTE redirection.
- `osmocom_installation/`: Contains scripts and subdirectories for installing various Osmocom components (bsc, bts, ggsn, hlr, etc.).

---

## High-Level Overview

This project is a research and testing platform for cellular network redirection attacks, specifically targeting LTE and 2G (EGPRS) networks. It automates the setup of a multi-component environment using Docker containers, SDR (Software Defined Radio) hardware, and open-source telecom stacks (Osmocom, OpenLTE, Asterisk). The system is designed to:

- Deploy fake base stations (BTS) for 2G and LTE.
- Redirect mobile devices to insecure or controlled networks.
- Provide a VoIP core (Asterisk) for call handling.
- Support multiple SDR hardware types (BladeRF, LimeSDR, USRP, etc.).
- Automate network, firewall, and hardware configuration for rapid testing.

**Intended Use:**  
This platform is for research, education, and lawful security testing only. It requires root privileges and direct hardware access.

---

## Prerequisites & Environment Setup

### System Requirements

- **Operating System:** Ubuntu 22.04 (tested)
- **Privileges:** Must be run as root (sudo)
- **Hardware:** Supported SDR (BladeRF-xA4, LimeSDR, USRP B200/ANTSDR)
- **Internet Access:** Required for package installation and Docker image pulls

### Dependencies

The following packages will be installed automatically by `build.sh`:
- `chromium-browser`
- `docker.io`, `docker-compose-v2`, `docker-buildx`
- `kconfig-frontends`
- `xterm`, `shellinabox`, `tmux`
- `libbladerf2`

### Environment Preparation

- Ensure your SDR hardware is connected.
- Back up any existing `.config` file in the project root (it will be renamed to `.config.old`).
- The build process will prompt for hardware selection and generate a `.config` file.
- Docker and network services will be restarted during setup.

---

## Step-by-Step Usage Instructions

### 1. Running `sudo ./build.sh`

**Purpose:**  
Prepares the environment, installs dependencies, configures hardware, resets firewall rules, and builds all required Docker containers.

#### What Happens Step-by-Step

1. **Install Dependencies:**  
   Installs all required system packages for SDR, Docker, and supporting tools.

2. **Backup Configuration:**  
   If a `.config` file exists, it is renamed to `.config.old` to preserve previous settings.

3. **Set Working Directory:**  
   The script determines its own directory and sets `$MYPATH` for consistent path usage.

4. **Reset Firewall Rules:**  
   Runs `reset_tables.sh` to flush and reset all iptables rules, ensuring a clean network state.

5. **Restart Docker & Network:**  
   - Restarts the Docker service.
   - Releases and renews the DHCP lease to ensure proper network configuration.

6. **Hardware Selection & Detection:**  
   - Runs `scripts/check_hw.sh`, which uses a menu-driven interface (`kconfig-mconf builder/selected_stuff`) to let you select your SDR hardware and features (Osmocom, OpenLTE, Asterisk).
   - Generates a `.config` file with your selections.

7. **(Conditional) Build Hardware-Specific Containers:**  
   - If the environment variable `REPONSE` is set to 1, builds and starts Docker containers in `builder/` and `builder/bladerf/`.

8. **Build and Start Core Services:**  
   - Runs `osmocom.sh` to build and start all Osmocom network components (core, HLR, MGW, STP, GGSN, SGSN, MSC, BSC, SIP, TRX, BTS, PCU) as Docker containers.
   - Runs `asterisk.sh` to build and start the Asterisk VoIP server as a Docker container.
   - Runs `openlte.sh` to build and start the OpenLTE redirector as a Docker container.

**User Prompts:**  
- You will interact with a hardware selection menu.
- No other user input is required during `build.sh`.

**Expected Outcome:**  
- All required containers are built and running.
- Network and firewall are reset and configured.
- The system is ready for runtime configuration and attack simulation via `run.sh`.

### 2. Running `sudo ./run.sh`

**Purpose:**  
Launches the full attack/test environment, configures network interfaces, starts all required containers, and opens interactive terminals for 2G, LTE redirection, and Asterisk.

#### What Happens Step-by-Step

1. **Root Privilege Check:**  
   Ensures the script is run as root. Exits if not.

2. **Set Working Directory:**  
   Sets `$MYPATH` to the script's directory for consistent path usage.

3. **Run SRSRAN Performance Script:**  
   Executes `srsran_performance` (purpose: likely to benchmark or check SDR performance).

4. **Prompt for Forwarding Interface (Device):**  
   Asks the user to enter the network interface to use for forwarding (e.g., `enp3s0`, `eth0`).

5. **Configure NAT for EPC:**  
   Runs `srsepc_if_masq.sh <interface>` to enable IP forwarding and set up NAT (masquerading) for the selected interface.

6. **Prompt for Forwarding Interface (IP):**  
   Asks the user to enter the IP address for the forwarding interface.

7. **Update Osmocom Configs:**  
   - Updates `osmo_egprs/configs/osmo-bsc.cfg` to set the remote IP.
   - Updates `osmo_egprs/configs/osmo-sgsn.cfg` to set the listen IP.

8. **Prompt to Restart Docker:**  
   Asks if Docker should be restarted. Defaults to "Y" (yes).

9. **Restart Docker (if chosen):**  
   Restarts the Docker service if the user agrees.

10. **Trigger Udev and Choose Interface:**  
    - Triggers udev events to ensure device nodes are up to date.
    - Runs `choose_interface.sh` to select and propagate the chosen interface configuration to all relevant locations.

11. **Load GTP Kernel Module:**  
    Loads the GTP (GPRS Tunneling Protocol) kernel module for mobile core networking.

12. **Stop Udev Services:**  
    Stops udev and related systemd sockets to avoid conflicts.

13. **Start Docker Containers:**  
    - Starts Osmocom EGPRS containers (`osmo_egprs/`).
    - Starts LTE redirection containers (`redirect_4_2g/`).
    - Starts Asterisk VoIP container (`asterisk/`).

14. **Open Interactive Terminals:**  
    - Opens a terminal for 2G base station (`2G.sh`).
    - Opens a terminal for LTE redirection (`redir.sh`).
    - Opens a terminal for Asterisk (`asterisk.sh`).

15. **Set DNS Resolver:**  
    Sets `/etc/resolv.conf` to use Google DNS (8.8.8.8).

16. **Enable Forwarding and NAT:**  
    - Runs `forward.sh` to enable IP forwarding and set up additional NAT rules for interfaces `enp114s0` and `apn0`.

17. **Re-run NAT for EPC:**  
    Runs `srsepc_if_masq.sh enp114s0` to ensure NAT is set up for the main interface.

18. **Open Telnet Session:**  
    Opens a telnet connection to `172.17.0.2` on port `30001` (likely to interact with a running service or container).

**User Prompts:**  
- Forwarding interface (device name)
- Forwarding interface (IP address)
- Whether to restart Docker

**Expected Outcome:**  
- All containers and services are running.
- Network interfaces and NAT are configured.
- Interactive terminals are open for 2G, LTE, and Asterisk.
- The system is ready for attack simulation or research.

---

## Troubleshooting & Common Issues

- **Script must be run as root:**  
  Both `build.sh` and `run.sh` require root privileges. Always use `sudo`.

- **Docker or network errors:**  
  Ensure Docker is installed and running. The scripts will attempt to restart Docker, but check `systemctl status docker` if issues persist.

- **SDR hardware not detected:**  
  Make sure your SDR is connected before running `build.sh`. Use `lsusb` to verify hardware presence.

- **Firewall/NAT issues:**  
  The scripts reset and configure iptables. If you have custom firewall rules, back them up before running.

- **Missing dependencies:**  
  `build.sh` installs all required packages, but if you see missing command errors, rerun `build.sh` or install the package manually.

- **Interactive terminals do not open:**  
  The scripts use `gnome-terminal`. If you use a different desktop environment, modify the script to use your terminal emulator.

- **Configuration not applied:**  
  If changes to `.config` or hardware selection are not reflected, delete `.config` and rerun `build.sh`.

---

## Additional Notes

- **Security Warning:**  
  This project is for research and lawful testing only. Running fake base stations is illegal in many jurisdictions without proper authorization.

- **Stopping the Environment:**  
  Use `docker compose down` in each service directory to stop containers. Reboot to reset network and firewall changes.

- **Logs and Debugging:**  
  - Docker logs: `docker logs <container_name>`
  - Service logs: Check within each running terminal or container.

- **Extending the System:**  
  - Add new SDR hardware by editing `builder/selected_stuff` and `scripts/check_hw.sh`.
  - Add new network services by creating new Dockerfiles and compose files in the appropriate directories.

- **Further Reading:**  
  See the `README_old.md` for additional context and manual steps for advanced use. 
