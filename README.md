# Zero Trust Subinterface Configuration

## Project Overview
This project demonstrates a high-security network architecture typically found in Enterprise environments. Unlike standard enterprise networks that utilize Layer 3 switches for inter-VLAN routing, this topology utilizes a **Palo Alto Next-Generation Firewall (NGFW)** as the gateway for all internal subnets (Router-on-a-Stick).

The goal is to move beyond simple connectivity and establish a **Zero Trust** foundation. By forcing all traffic between internal departments to pass through the firewall, the network ensures comprehensive **Deep Packet Inspection (DPI)**, strict logging, and granular policy enforcement between security zones.

## Network Topology & Enclaves

The network is segmented into three distinct security zones, isolated by Layer 2 VLANs on a Cisco switch and routed via Layer 3 Subinterfaces on the Palo Alto Firewall.

| Zone / Enclave | VLAN ID | Description |
| :--- | :--- | :--- | :--- |
| **Corporate** | 10 | Standard staff network |
| **Classified** | 20 | Sensitive Network  |
| **Guest** | 30 | Non-employee guest traffic |

## Traffic Flow Architecture

1.  **Ingress (Layer 2):** Traffic originates from an endpoint (e.g., Corporate PC). The Cisco Switch tags the frame with `VLAN 10` and forwards it via a generic 802.1Q Trunk port to the Firewall.
2.  **Gateway Processing (Layer 3):** The Palo Alto Firewall receives the tagged frame on `ethernet1/1.10`. It acts as the Default Gateway for the subnet.
3.  **Inspection Engine:** Unlike a Layer 3 switch which only checks IP headers, the Palo Alto inspects the payload. It checks:
    *   **App-ID:** Is the traffic actually what it claims to be?
    *   **User-ID:** Who is sending the packet?
    *   **Security Policy:** Is the Corporate Zone allowed to talk to Classified Zone?
4.  **Egress:** If the traffic matches an "Allow" policy, the Firewall routes the packet to the destination subnet (e.g., `ethernet1/1.30`), re-tags it for `VLAN 30`, and sends it back down the trunk to the switch for delivery.

## Technology Stack
*   **Virtualization:** GNS3 Network Simulator
*   **Firewall:** Palo Alto VM-Series (PA-VM)
*   **Switching:** Cisco IOSvL2
*   **Endpoints:** Ubuntu Docker Containers

## Project Evidence & Gallery

### 1. Network Topology
*A visual overview of the GNS3 layout, showing the relationship between the separate Enclaves, the Cisco Access Switch, and the Palo Alto Gateway.*
> **[Insert Screenshot: GNS3 Topology Map]**

### 2. Switch VLANs


### 2. Interface Configuration (Subinterfaces)
*Evidence of the "Router-on-a-Stick" configuration. This shows the physical interface divided into logical subinterfaces, each assigned to a specific VLAN ID, IP subnet, and Security Zone.*
> **[Insert Screenshot: Palo Alto Network > Interfaces Tab]**

### 3. Security Policy Implementation
*The "Zero Trust" mechanism. By default, the firewall drops all inter-zone traffic. This screenshot demonstrates the explicit rules created to allow specific, monitored communication between the Corporate and Restricted zones.*
> **[Insert Screenshot: Palo Alto Policies > Security Tab]**

### 4. Traffic Inspection & Verification
*Proof of segmentation. The logs demonstrate the initial default "Deny" action (blocking unauthorized lateral movement) followed by a successful "Allow" action after policy remediation.*
> **[Insert Screenshot: Palo Alto Monitor > Traffic Logs showing both Red (Deny) and Green (Allow) entries]**