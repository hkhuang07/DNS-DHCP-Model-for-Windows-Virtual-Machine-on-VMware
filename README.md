<div align="center">
  <img src="https://img.shields.io/badge/Project-Network%20Administration-blue?style=for-the-badge&logo=networkmanager&logoColor=white" alt="Project Badge"/>
  <img src="https://img.shields.io/badge/Services-DNS%20%26%20DHCP-success?style=for-the-badge&logo=dns&logoColor=white" alt="Services Badge"/>
  <img src="https://img.shields.io/badge/Environment-VMware%20Workstation%20Pro-critical?style=for-the-badge&logo=vmware&logoColor=white" alt="VMware Badge"/>
  <img src="https://img.shields.io/badge/OS-Windows%20Server%202012%20R2-informational?style=for-the-badge&logo=windows&logoColor=white" alt="Windows Server Badge"/>
</div>

# 🌐 Network Administration Project: DNS and DHCP Deployment on VMware

## ✨ Project Overview

This project details the process of deploying and configuring essential network services, **DNS** (Domain Name System) and **DHCP** (Dynamic Host Configuration Protocol), within a **VMware Workstation Pro** virtualized environment. The model includes a **Windows Server 2012 R2** machine acting as both DNS and DHCP Server, along with client machines running **Windows 7**, **Windows 2000 Professional**, and **Windows 2003** to test dynamic IP allocation and domain name resolution capabilities. The goal is to build a stable, manageable, and scalable internal network infrastructure.

---

## 📐 I. Model Analysis and Design

### 1. Requirements Analysis

#### 1.1 Technical Requirements

* **Subnetting:**
    * Total devices: 21 machines (20 employee machines + 1 administrator machine).
    * Total IPs needed: 23 (including Gateway and Broadcast).
    * To accommodate 23 IPs, we need $2^n > 23 \implies n = 5$ bits for the host portion.
    * Number of network bits in Subnet Mask: $32 - 5 = 27$.
    * **Suitable CIDR:** `/27` (Subnet Mask: `255.255.255.224`).
    * **Available IP range:** `192.168.250.10 – 192.168.250.29`.
    * **Gateway:** `192.166.250.1`.
    * **DNS Server:** `192.168.250.2`.

* **<img src="https://img.icons8.com/color/20/000000/dns.png"/> DNS (Domain Name System):**
    * **Main function:** Converts domain names into IP addresses, making network device access easier.
    * **Supports internal domain name management and links with other services.**
    * **Supports basic DNS records:**
        * `A` (Address): Links a domain name to an IP address.
        * `MX` (Mail Exchange): Points to an email server.
        * `CNAME` (Canonical Name): An alias for another domain.
        * `PTR` (Pointer Record): Converts an IP address to a domain name (Reverse DNS).
    * **DNS server hierarchy:**
        * `Primary DNS Server`: Manages and stores primary data.
        * `Secondary DNS Server`: Provides redundancy and replicates data from the Primary to ensure availability.

* **<img src="https://img.icons8.com/color/20/000000/dhcp-server.png"/> DHCP (Dynamic Host Configuration Protocol):**
    * **Main function:** Automatically assigns IP addresses and other network configuration parameters to end devices on the network.
    * **Supports additional information:**
        * `Subnet Mask`: `255.255.255.224` (consistent with /27).
        * `Gateway`: `192.168.250.1`.
        * `DNS Server`: `192.168.250.2`.
    * **IP Pool Management:** Allocates IP addresses within an optimal range for devices (20 employee computers + 1 system administration machine).

#### 1.2 Hardware Requirements (Virtual Machines)

* **<img src="https://img.icons8.com/color/20/000000/server.png"/> Servers:**
    * Operating system: **Windows Server 2012 R2** running DNS and DHCP services.
    * Network parameters:
        * IP Address: `192.168.250.2` (Static IP).
        * Default Gateway: `192.168.250.1`.
        * Preferred DNS Server: `192.168.250.2`.
* **<img src="https://img.icons8.com/color/20/000000/laptop.png"/> End Devices (Clients):**
    * Operating systems:
        * Windows 2000 Professional
        * Windows 7
    * Network parameters: Obtain dynamic IPs from DHCP Server, Preferred DNS Server: `192.168.250.2`.

#### 1.3 Software Requirements

* Server operating system: **Windows Server 2012 R2**.
* Tools: Server Manager for DNS and DHCP installation and configuration.
* Virtualization environment: **VMware Workstation Pro**.

### 2. Model Design

#### 2.1 Overall Network Structure

* The network uses a private Class C IP address range, `192.168.250.0/27`.
* **Subnet mask:** `255.255.255.224`.
    * Host bits: `32 – 27 = 5`.
    * Manageable hosts: $2^5 – 2 = 30$ (30 addresses available, sufficient for the 23 required IPs).

#### 2.2 Description

* The Windows Server 2012 R2 machine will perform two main roles:
    * Configure and manage the DNS Server with the enterprise domain name (`a.local`).
    * Configure the DHCP Server to assign dynamic IPs to client machines.
* End devices will automatically obtain IP addresses from the DHCP Server and use the DNS Server for domain name resolution.

#### 2.3 Network Diagram

* **Server (DNS + DHCP):** `192.168.250.2`
* **Gateway:** `192.168.250.1`
* **Client:** Dynamic IP range from `192.168.250.10 – 192.168.250.29`

*(Network diagram will be displayed as an image in the project's root folder or within the README, e.g., `images/network_diagram.png`)*

### 3. Detailed DNS Model

#### 3.1 Zone Partitioning

* **Primary Zone:**
    * Main enterprise domain name: `a.local` (or `group01networkadministrator.local`).
    * Stores all primary DNS records.
* **Reverse Lookup Zone:**
    * Creates PTR records to map from IP to domain name, enabling reverse resolution.

#### 3.2 Record Management

* **A Record (Address):**
    * `server.a.local` $\rightarrow$ `192.168.250.2`
* **CNAME Record (Canonical Name):**
    * `www` $\rightarrow$ `server1.local` (alias for `server1.local` if it exists, otherwise it will be an alias for `server.a.local` or the DNS server's hostname).
* **MX Record (Mail Exchanger):**
    * `mail.local` $\rightarrow$ `192.168.250.3` (points to the mail server, if any).
* **PTR Record (Pointer Record):**
    * `192.168.250.2` $\rightarrow$ `server1.local` (or `server.a.local`).

### 4. Detailed DHCP Model

#### 4.1 IP Pool Allocation

* **IP allocation range:** `192.168.250.10 – 192.168.250.29` (20 computers + 1 admin machine).

#### 4.2 Dynamic IP Configuration

* Install DHCP Role on Windows Server 2012 R2.
* **Create DHCP Scope:**
    * Scope Name: `Scope1`
    * Start IP: `192.168.250.10`
    * End IP: `192.168.250.29`
    * Subnet Mask: `255.255.255.224`
    * Default Gateway: `192.168.250.1`
    * DNS Server: `192.168.250.2`
    * Lease duration: 1 day.

---

## 🛠️ II. Deployment and Testing

This section details the steps for installing and configuring DNS and DHCP services on Windows Server 2012 R2, as well as how to test on client machines.

### 1. System Deployment

#### 1.1 DNS and DHCP Installation via Server Manager

##### 1.1.1 Network Configuration for Windows Server 2012 R2 Server

1.  <img src="https://img.icons8.com/color/20/000000/settings.png"/> Open **Network and Sharing Center** (Right-click the network icon on the Taskbar or via Control Panel).
2.  <img src="https://img.icons8.com/color/20/000000/network-card.png"/> Select **"Change adapter settings"**.
3.  <img src="https://img.icons8.com/color/20/000000/ethernet-cable.png"/> Right-click **"Ethernet"** (or corresponding network card name) and select **"Properties"**.
4.  <img src="https://img.icons8.com/color/20/000000/ipv4.png"/> Select **"Internet Protocol Version 4 (TCP/IPv4)"** and click **"Properties"**.
5.  <img src="https://img.icons8.com/color/20/000000/ip-address.png"/> Select **"Use the following IP address"** and assign a static IP:
    * **IP Address:** `192.168.250.2`
    * **Subnet Mask:** `255.255.255.224`
    * **Default Gateway:** `192.168.250.1`
    * **Preferred DNS Server:** `192.168.250.2`
6.  <img src="https://img.icons8.com/color/20/000000/ok.png"/> Click **OK** to save the configuration.

##### 1.1.2 DNS Server Installation

1.  <img src="https://img.icons8.com/color/20/000000/server.png"/> **Open Server Manager:** Search for and select "Server Manager" from the Start Menu or Taskbar.
2.  <img src="https://img.icons8.com/color/20/000000/plus.png"/> **Add DNS Server Role:**
    * In Server Manager, select **"Manage"** $\rightarrow$ **"Add Roles and Features"**.
    * Click **"Next"** repeatedly until the **"Server Roles"** screen.
    * Check **"DNS Server"**.
    * Click **"Add Features"** when prompted.
    * Continue clicking **"Next"** and finally select **"Install"** to begin the installation.
3.  <img src="https://img.icons8.com/color/20/000000/dns.png"/> **Create Forward Lookup Zone:**
    * After DNS Server installation, in Server Manager $\rightarrow$ **"Tools"** $\rightarrow$ **"DNS"** to open DNS Manager.
    * In DNS Manager, right-click **"Forward Lookup Zones"** and select **"New Zone..."**.
    * In the New Zone Wizard, click **"Next"**.
    * Select **"Primary Zone"** $\rightarrow$ **"Next"**.
    * Enter the main domain name for the Zone: **`a.local`** (or `group01networkadministrator.local`) $\rightarrow$ **"Next"**.
    * Select **"Create a new file with this file name:"** (default file name) $\rightarrow$ **"Next"**.
    * Choose **"Do not allow dynamic updates"** (or an option suitable for security requirements) $\rightarrow$ **"Next"**.
    * Click **"Finish"** to complete Zone creation.
4.  <img src="https://img.icons8.com/color/20/000000/active-directory.png"/> **Configure Server as Domain Controller (optional, if Active Directory is desired):**
    * After installing the DNS role, in Server Manager, you will see a yellow flag notification in the top right corner. Click it and select **"Promote this server to a domain controller"**.
    * In Deployment Configuration:
        * Select **"Add a new forest"**.
        * Enter **Root domain name:** `a.local` $\rightarrow$ **"Next"**.
    * In Domain Controller Options:
        * Check **"Domain Name System (DNS) server"**.
        * Select Forest functional level and Domain functional level (default is Windows Server 2012 R2).
        * Enter a password for Directory Service Restore Mode (DSRM) $\rightarrow$ **"Next"**.
    * At DNS Options: Click **"Next"** (there may be a warning about delegation, click **"Yes"** to proceed).
    * At Additional Options: Confirm the default NetBIOS Domain Name (`A`) $\rightarrow$ **"Next"**.
    * Path: Keep default paths (Database Folder, Log Files Folder, SYSVOL Folder) $\rightarrow$ **"Next"**.
    * Review Options: Review the settings $\rightarrow$ **"Next"**.
    * Prerequisites Check: Wait for the system to check. If there are no errors, click **"Install"**.
    * After installation is complete, the server will require a **restart**. Restart the server.
5.  <img src="https://img.icons8.com/color/20/000000/domain.png"/> **Join client machines to the Domain (on client machines):**
    * **Configure DNS on client machines:** Ensure client machines are using the DNS Server IP `192.168.250.2`.
    * Open **System Properties**: Press `Window + R`, type `sysdm.cpl`, and press `OK`.
    * In the **"Computer Name"** tab (or "Network Identification"), click **"Change..."**.
    * Select **"Domain"** and enter your domain name: `a.local` $\rightarrow$ **"OK"**.
    * When prompted, enter the domain Administrator account information (e.g., `Administrator` and the password set for the Administrator account on the Domain Controller).
    * If successful, you will receive a "Welcome to the a.local domain" message. Click **OK**, then restart the client machine.

##### 1.1.3 DHCP Server Installation

1.  <img src="https://img.icons8.com/color/20/000000/server.png"/> **Add DHCP Server Role:**
    * In Server Manager, select **"Manage"** $\rightarrow$ **"Add Roles and Features"**.
    * Click **"Next"** repeatedly until the **"Server Roles"** screen.
    * Check **"DHCP Server"**.
    * Click **"Add Features"** when prompted.
    * Continue clicking **"Next"** and finally select **"Install"** to begin the installation.
2.  <img src="https://img.icons8.com/color/20/000000/dhcp-server.png"/> **Configure DHCP Scope:**
    * After DHCP Server installation, in Server Manager, click the yellow flag notification in the top right corner and select **"Complete DHCP configuration"**.
    * In DHCP Manager (Server Manager $\rightarrow$ **"Tools"** $\rightarrow$ **"DHCP"**):
        * Navigate to: `<server name>` $\rightarrow$ `IPv4`.
        * Right-click `IPv4` and select **"New Scope..."**.
        * In the New Scope Wizard, click **"Next"**.
        * **Scope Name:** `Scope1` $\rightarrow$ **"Next"**.
        * **IP Address Range:**
            * Start IP: `192.168.250.10`
            * End IP: `192.168.250.29`
            * Subnet Mask: `255.255.255.224`
            * Click **"Next"**.
        * Add Exclusions and Delay (if any IPs need to be excluded from the Pool) $\rightarrow$ **"Next"**.
        * Lease Duration: Select `1 day` $\rightarrow$ **"Next"**.
        * Configure DHCP Options: Select **"Yes, I want to configure these options now"** $\rightarrow$ **"Next"**.
        * **Router (Default Gateway):** Enter `192.168.250.1` $\rightarrow$ **"Add"** $\rightarrow$ **"Next"**.
        * **DNS Servers:** Ensure the DNS Server IP `192.168.250.2` is present or add it $\rightarrow$ **"Next"**.
        * WINS Servers (Skip if not used) $\rightarrow$ **"Next"**.
        * Activate Scope: Select **"Yes, I want to activate this scope now"** $\rightarrow$ **"Next"**.
        * Click **"Finish"** to complete Scope creation.

### 2. Service Verification and Startup

#### 2.1 Test DNS using `nslookup` and `ping` commands

1.  <img src="https://img.icons8.com/color/20/000000/laptop.png"/> **On client machines (Windows 7 and Windows 2000 Professional):**
    * Open Command Prompt: Press `Window + R`, type `cmd`, and press `Enter`.
2.  <img src="https://img.icons8.com/color/20/000000/command-line.png"/> **Test domain name resolution with `nslookup`:**
    * Type command: `nslookup`
    * At the `>` prompt, type the domain name: `a.local` (or `group01networkadministrator.local`)
    * Check if it resolves correctly to IP `192.168.250.2`.
3.  <img src="https://img.icons8.com/color/20/000000/ping.png"/> **Test connectivity to DNS server with `ping`:**
    * Type command: `ping 192.168.250.2`
    * Ensure you receive replies (Reply from...) to confirm connectivity.

#### 2.2 Test DHCP by configuring clients to automatically obtain IP

1.  <img src="https://img.icons8.com/color/20/000000/laptop.png"/> **On client machines (Windows 7 and Windows 2000 Professional):**
    * **Configure automatic IP obtainment:**
        * **Windows 2000 Professional:** Right-click My Network Place $\rightarrow$ Properties $\rightarrow$ Right-click Local Area Connection $\rightarrow$ Properties $\rightarrow$ Select Internet Protocol (TCP/IP) $\rightarrow$ Properties $\rightarrow$ Select **"Obtain an IP address automatically"**. Enter Preferred DNS server: `192.168.250.2`.
        * **Windows 7:** Click Network on Desktop $\rightarrow$ Properties $\rightarrow; Change adapter settings $\rightarrow$ Right-click Ethernet $\rightarrow$ Properties $\rightarrow$ Select Internet Protocol Version 4 (TCP/IPv4) $\rightarrow$ Properties $\rightarrow$ Select **"Obtain an IP address automatically"**. Enter Preferred DNS server: `192.168.250.2`.
        * Click **OK** to save the configuration.
    * Open Command Prompt: Press `Window + R`, type `cmd`, and press `Enter`.
    * **Release old IP:** Type command: `ipconfig /release`
    * **Renew IP from DHCP:** Type command: `ipconfig /renew`
2.  <img src="https://img.icons8.com/color/20/000000/ip-address.png"/> **Verify obtained IP:**
    * Type command: `ipconfig /all`
    * Ensure the obtained IP is within the range `192.168.250.10 – 192.168.250.29`.
3.  <img src="https://img.icons8.com/color/20/000000/dhcp-server.png"/> **Verify on DHCP Server interface (on Windows Server 2012 R2):**
    * Open DHCP Manager.
    * Navigate to `IPv4` $\rightarrow$ `Scope1` $\rightarrow$ `Address Leases`.
    * You should see a list of IP addresses assigned to client machines, along with their hostnames and lease times.

### 3. Troubleshooting

#### 3.1 Check network connectivity, DNS/DHCP services, and event logs on Server Manager.

* <img src="https://img.icons8.com/color/20/000000/network-connection.png"/> **Check network connectivity:**
    * Run `ping` command to test connectivity between server and client machines.
    * Ping from server to client machines (e.g., `ping 192.168.250.10`).
    * Ping from client machines to server (e.g., `ping 192.168.250.2`).
* <img src="https://img.icons8.com/color/20/000000/services.png"/> **Check DNS/DHCP services:**
    * Open Server Manager $\rightarrow$ **"Tools"** $\rightarrow$ **"DNS"** or **"DHCP"**.
    * In the DNS/DHCP management console, ensure services are running. If a service is stopped (red icon or "Stopped" status), right-click it and select "Start" or "Restart".
* <img src="https://img.icons8.com/color/20/000000/event-log.png"/> **View Event Logs:**
    * Open Run dialog: Press `Window + R`, type `eventvwr`, and press `Enter`.
    * In "Event Viewer", navigate to **"Windows Logs"** $\rightarrow$ **"System"** or **"Application"**.
    * Check logs related to DNS (Event ID 4000-4001 for DNS Server) and DHCP (Event ID 1000-1002 for DHCP Server) for any errors or warnings.

---

## ✅ III. Conclusion and Future Development

### 1. Conclusion

The DNS and DHCP system has been successfully deployed and configured, effectively meeting the requirements for domain name management and dynamic IP allocation. The network is well-segmented, ensuring stable operation and scalability.

### 2. Future Development

* <img src="https://img.icons8.com/color/20/000000/update.png"/> Upgrade the system to newer Windows Server versions to leverage advanced features and security.
* <img src="https://img.icons8.com/color/20/000000/integration.png"/> Integrate additional essential network services such as Active Directory, Web Server (IIS), File Server, or VPN to build a more comprehensive infrastructure.
* <img src="https://img.icons8.com/color/20/000000/expand.png"/> Expand the network scale to support multiple departments or new branches, potentially including VLANs and Routing.
