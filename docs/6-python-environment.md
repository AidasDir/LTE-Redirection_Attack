# Python Environment

Relevant source files 
* [`init_pyenv.sh`](../init_pyenv.sh)
* [`scripts/telnet_free.py`](../scripts/telnet_free.py)

This document details the Python environment used within the LTE-Redirection Attack system. It covers the setup of the required Python environment and the Python scripts that are used to configure and control various components of the attack system, with a focus on the telnet configuration scripts. For information about running the complete attack process, see [Running the Attack](running-the-attack.md), and for overall system architecture, see [System Architecture](4-system-architecture.md).

## Environment Setup

The Python environment for this system is based on Python 3.7 and is initialized through a dedicated setup script. This ensures consistent operation across different systems where the attack might be deployed.

The environment initialization process includes:

1. Downloading pip for Python 2.7 (legacy requirement)
2. Installing Miniconda3 for environment management
3. Creating a dedicated Python 3.7 environment named `py37`
4. Configuring shell integration
5. Activating the environment for use

```
# Example steps from init_pyenv.sh
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
bash Miniconda3-latest-Linux-x86_64.sh
conda create -n py37 python=3.7
conda init bash
conda activate py37
```

*Sources: [`init_pyenv.sh`](../init_pyenv.sh)*

## Telnet Configuration Scripts

The primary Python components in this system are the telnet configuration scripts that configure the LTE redirector container. These scripts connect to the redirector container via telnet and send configuration commands to set up the fake base station parameters.

### Script Structure and Operation

The telnet scripts use an asynchronous approach with the `asyncio` and `telnetlib3` libraries to communicate with the redirector container.

*Sources: [`scripts/telnet_free.py`](../scripts/telnet_free.py)*

### Configuration Parameters

The scripts configure several key parameters of the LTE redirector:

| Parameter Category  | Example Parameters           | Purpose                            |
| ------------------- | ---------------------------- | ---------------------------------- |
| Cell Identification | n_id_cell, cell_id           | Uniquely identify the fake cell    |
| Network Identity    | MCC, MNC                     | Define the mobile network operator |
| Radio Configuration | band, dl_earfcn              | Set frequency parameters           |
| Hardware Settings   | rx_gain, tx_gain             | Configure SDR hardware             |
| System Parameters   | tracking_area_code, n_ant    | Additional network parameters      |

*Sources: [`scripts/telnet_free.py`](../scripts/telnet_free.py)*

### Operator-Specific Scripts

The system includes multiple telnet scripts, each configured for a specific mobile network operator:

* [`telnet_free.py`](../scripts/telnet_free.py) - For Free Mobile (MCC: 208, MNC: 15)
* [`telnet_orange.py`](../scripts/telnet_orange.py) - For Orange
* [`telnet_sfr.py`](../scripts/telnet_sfr.py) - For SFR
* [`telnet_bouygues.py`](../scripts/telnet_bouygues.py) - For Bouygues Telecom

Each script contains operator-specific parameters while following the same communication pattern.

*Sources: [`scripts/telnet_free.py`](../scripts/telnet_free.py)*

## Telnet Configuration Workflow

The telnet configuration scripts follow a consistent workflow pattern:

Configuration commands include:

1. Setting random cell identifiers (`n_id_cell` and `cell_id`)
2. Enabling configuration file usage (`use_cnfg_file 1`)
3. Setting antenna configuration (`n_ant 2`)
4. Configuring network identity (`mcc 208`, `mnc 15` for Free Mobile)
5. Setting frequency parameters (`band 7`, `dl_earfcn 3350`)
6. Setting tracking area code (`tracking_area_code 6601`)
7. Configuring hardware parameters (`rx_gain 30`, `tx_gain 80`)

*Sources: [`scripts/telnet_free.py`](../scripts/telnet_free.py)*

## Integration with System Components

The Python environment integrates with the rest of the attack system through the shell scripts that call the Python scripts based on the selected mobile operator.

*Sources: [`scripts/telnet_free.py`](../scripts/telnet_free.py) [`init_pyenv.sh`](../init_pyenv.sh)*

## Python Dependencies

The Python scripts in this system rely on the following key dependencies:

| Dependency | Purpose                                                                |
| ---------- | ---------------------------------------------------------------------- |
| asyncio    | Provides asynchronous I/O framework for handling concurrent operations |
| telnetlib3 | Enables telnet connections for configuring the redirector container    |
| random     | Generates random values for network parameters to avoid collisions     |

*Sources: [`scripts/telnet_free.py`](../scripts/telnet_free.py)*

## Typical Script Execution Flow

A typical execution of the telnet configuration scripts follows this pattern:

1. The script is invoked by the container launcher script based on operator selection
2. The script establishes a telnet connection to the redirector container at 172.17.0.2:30000
3. It reads the initial response from the connection
4. It generates random values for the cell identifiers
5. It sends a series of configuration commands, reading the response after each command
6. After all parameters are configured, the connection is closed

This process ensures that the LTE redirector is properly configured for the specific mobile operator before the attack is launched.

*Sources: [`scripts/telnet_free.py`](../scripts/telnet_free.py)*

*Source: [DeepWiki LTE-Redirection_Attack Python Environment](https://deepwiki.com/AidasDir/LTE-Redirection_Attack/6-python-environment)*
