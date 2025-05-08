# System Architecture

This document provides a comprehensive overview of the LTE-Redirection Attack system architecture, detailing the major components and their interactions. It explains the structure of the Docker containers, the communication flow between components, and how the system orchestrates the cellular network redirection attack from LTE to 2G.

For specific information about running the attack, see [Running the Attack](3-running-the-attack.md), and for setting up the environment, refer to [Setup and Installation](2-setup-and-installation.md).

## 1. Architectural Overview

![](System-Architecture-1.png)

The LTE-Redirection Attack system is designed as a multi-container Docker application that simulates cellular network components to execute a downgrade attack from 4G LTE to 2G/EGPRS. The system consists of three primary Docker containers that work in concert, controlled by orchestration scripts.

### System Components Diagram

![](System-Architecture-2.png)

## 2. Container Architecture

![](System-Architecture-3.png)

The system uses three specialized Docker containers, each responsible for a specific part of the cellular network attack:

### 2.1 LTE Redirector (redirect_4_2g)

![](System-Architecture-4.png)

The LTE redirector container implements a fake LTE base station (eNodeB) using a modified version of OpenLTE. This component broadcasts selected Mobile Country Code (MCC) and Mobile Network Code (MNC) to target specific networks and configures redirection parameters to force devices to downgrade to 2G.

Key modifications in the OpenLTE code allow for:

* Modifying RRC connection release messages to include redirection information
* Handling tracking area update requests with rejections
* Setting appropriate EMM (EPS Mobility Management) cause values

### 2.2 2G/EGPRS Network (osmo_egprs)

![](System-Architecture-5.png)

The osmo_egprs container implements a complete Osmocom-based 2G network stack, including Home Location Register (HLR), Mobile Switching Center (MSC), Base Station Controller (BSC), Serving GPRS Support Node (SGSN), and Gateway GPRS Support Node (GGSN). This provides complete cellular functionality and data services through GTP tunneling.

### 2.3 VoIP Component (asterisk)

![](System-Architecture-6.png)

The asterisk container implements an Asterisk PBX for call handling. It integrates with the Osmocom voice stack through the Media Gateway (MGW) to enable voice call interception and manipulation.

## 3. Communication Flow

![](System-Architecture-7.png)

The system implements a specific communication flow to execute the LTE to 2G redirection attack:

### Attack Flow Sequence Diagram

![](System-Architecture-8.png)

## 4. Technical Implementation Details

### 4.1 RRC Redirection Mechanism

The core of the attack is implemented through modifications to the OpenLTE codebase, particularly in the Radio Resource Control (RRC) message handling. The system modifies the `liblte_rrc.cc` file to include redirection information in the RRC Connection Release message:

### 4.2 MME Implementation

The system extends the MME (Mobility Management Entity) implementation to handle tracking area update requests with rejections, forcing devices to connect to the 2G network. Key modifications include:

| MME Procedure/State Additions           | Purpose                                                |
| --------------------------------------- | ------------------------------------------------------ |
| LTE_FDD_ENB_MME_PROC_TAU_REQUEST  | Define new procedure for tracking area update requests |
| LTE_FDD_ENB_MME_STATE_TAU_REJECT  | Define new state for rejecting tracking area updates   |
| send_tracking_area_update_request() | Handle incoming tracking area update requests          |
| send_tracking_area_update_reject()  | Send reject responses to force redirection             |

## 5. Orchestration Scripts

The system is controlled by several bash scripts that handle building, configuration, and running of the components:

### 5.1 Script Hierarchy and Relationships

### 5.2 Network Configuration

The system configures network interfaces, NAT, and routing to enable traffic forwarding between captured devices and the Internet:

## 6. SDR Hardware Integration

The system supports multiple Software Defined Radio (SDR) hardware options for both the LTE and 2G components:

| SDR Type         | Support Level | Component |
| ---------------- | ------------- | --------- |
| BladeRF-xA4      | Preferred     | LTE & 2G  |
| LimeSDR          | Alternative   | LTE & 2G  |
| USRP B200/ANTSDR | Alternative   | LTE & 2G  |

The SDR hardware integration is handled through the `LTE_fdd_enb_radio.cc` file, which includes modifications to support different SDR types and configurations, as well as specific settings for the BladeRF hardware.

## Summary

The LTE-Redirection Attack system implements a comprehensive cellular network security testing platform through a combination of Docker containers running modified OpenLTE and Osmocom components. The architecture enables the creation of fake LTE and 2G networks, with the capability to force mobile devices to downgrade from LTE to less secure 2G connections, where traffic can be intercepted and analyzed.

The system is highly modular and configurable, supporting multiple SDR hardware options and mobile network operators. The attack flow is orchestrated through a set of bash and Python scripts that automate the building, configuration, and execution of the attack components.

*Source: [DeepWiki LTE-Redirection_Attack System Architecture](https://deepwiki.com/AidasDir/LTE-Redirection_Attack/4-system-architecture)* 