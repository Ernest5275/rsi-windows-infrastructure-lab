# RSI Enterprise Windows Infrastructure Lab 🖥️🏢
**Academic Project, ECPI University (CYB360)** | Systems Administration | Dec 2025 – Jan 2026

## 📋 Overview
Over five weekly phases, I built a complete Windows Server environment for a fictional company, **RSI**, and its subsidiary **Guardian**, on a single Hyper-V host. The build covers virtualization, networking, a parent/child Active Directory forest, DHCP with failover, DNS, file shares, Group Policy, and Windows containers with Docker. Each phase was verified with PowerShell and documented in a weekly report for stakeholders.

## 🛠️ Technologies
`Windows Server 2019` `Hyper-V` `PowerShell` `Active Directory Domain Services` `DHCP (Failover)` `DNS` `Group Policy` `NTFS & Share Permissions` `Docker` `Windows Server Core Containers` `IIS`

## 🗺️ Network Diagram
```mermaid
flowchart TB
    subgraph HOST["Hyper-V Host · Windows Server 2019"]
        NAT["NAT: VC-RSI-LAN · 10.2.2.0/24"]
        SW["Internal Virtual Switch: VS-RSI-LAN"]
        NAT --- SW
        S1["RSI-Server-01 · 10.2.2.10<br/>Domain Controller · DHCP · DNS"]
        S2["RSI-Server-02 · 10.2.2.11<br/>DHCP Failover Partner"]
        S3["RSI-Server-03 · 10.2.2.12<br/>Server Core"]
        C1["RSI-Client-01 · Windows 11<br/>DHCP"]
        C2["RSI-Client-02 · Windows 11<br/>DHCP"]
        SW --- S1 & S2 & S3 & C1 & C2
        subgraph DOCKER["Docker · RSI-Net 192.168.1.0/24"]
            WEB["RSIWeb · IIS<br/>Host port 5678 → 80"]
            STD["RSIStd · Standard Server"]
        end
    end
```

## Phase 1: Hyper-V & Virtual Machine Configuration
- Installed the Hyper-V role and set separate folders for base images (`D:\Images`), virtual hard disks (`D:\VHD`), and VMs (`D:\VM`)
- Built three **base images** (Server GUI, Server Core, Windows 11) and created five VMs from **differencing disks** to save storage
- Applied a consistent naming standard (`RSI-Server-0X`, `RSI-Client-0X`, `-HD-0X.vhdx`)

| Base image | VMs built from it |
|---|---|
| RSI-GUI-Base.vhdx | RSI-Server-01, RSI-Server-02 |
| RSI-Core-Base.vhdx | RSI-Server-03 |
| RSI-W11-Base.vhdx | RSI-Client-01, RSI-Client-02 |

![Differencing disks](screenshots/p1-differencing-disks.png)

## Phase 2: Networking & Active Directory Prerequisites
- Created internal virtual switch **VS-RSI-LAN** and a **NAT** network (10.2.2.0/24) on the host
- Connected all VMs to the switch and assigned static IPs to the servers
- Installed the AD DS role on all three servers and activated Windows Server

![NAT configuration](screenshots/p2-netnat.png)

## Phase 3: Active Directory, DHCP, DNS & Shares
- Built a **parent/child forest**: `EF71RSI.local` and `GuardianDev.EF71RSI.local`
- Created OUs, security groups, and users for each department (Admin, HR, IT, Engineering, Operations) with a naming standard
- Joined both workstations to the domain in the **RSIWorkstations** OU
- Configured **DHCP with failover** (Hot Standby) between two servers, with scope options for router, DNS, and domain name
- Created departmental **file shares** with group-based share and NTFS permissions
- Added a DNS **reverse lookup zone** (2.2.10.in-addr.arpa)

![Parent and child domains](screenshots/p3-parent-child-domains.png)
![DHCP failover](screenshots/p3-dhcp-failover.png)
![RSI shares and permissions](screenshots/p3-rsi-shares-permissions.png)

## Phase 4: Group Policy & Docker
- **RSI-WS-Security GPO** (linked and enforced on RSIWorkstations): disables the built-in Administrator and Guest accounts, renames both, and uses Restricted Groups to control Remote Desktop Users
- **RSI-WS-Updates GPO**: automatic updates every Friday at 01:00 from an internal update server
- Installed Docker, pulled Windows Server Core, and built two reusable images: **web-image-template** (with the IIS Web Server role) and **std-image-template**
- Created host folders for container bind mounts

![Security GPO settings](screenshots/p4-gpo-ws-security.png)
![Docker images](screenshots/p4-docker-images.png)

## Phase 5: Finalizing Docker Containers
- Deployed **RSIWeb** from the web template, publishing container port 80 on host port 5678
- Deployed **RSIStd** from the standard template
- Created a separate Docker NAT network, **RSI-Net** (192.168.1.0/24), for the standard server
- Attached persistent **bind mounts**: `D:\RSIData1 → C:\webdata` (RSIWeb) and `D:\RSIData2 → C:\stddata` (RSIStd)

![Running containers](screenshots/p5-running-containers.png)
![Bind mount](screenshots/p5-rsiweb-bind-mount.png)

## 📸 All Screenshots
See the [screenshots](screenshots/) folder for every verification step.
<img width="1286" height="674" alt="p3-rsi-shares-permissions" src="https://github.com/user-attachments/assets/24a37731-47bf-4304-b432-6ad91f97a3f6" />
<img width="624" height="198" alt="p3-rsi-ous" src="https://github.com/user-attachments/assets/b0c7270c-5663-462b-8276-759d38ede47f" />
<img width="1284" height="528" alt="p3-rsi-groups" src="https://github.com/user-attachments/assets/09eb0b8f-34a8-4009-bc51-800753d5a03a" />
<img width="1095" height="253" alt="p3-parent-child-domains" src="https://github.com/user-attachments/assets/0fae8f2d-aa0a-45a7-b334-d84199c315a4" />
<img width="1030" height="145" alt="p3-dns-reverse-lookup" src="https://github.com/user-attachments/assets/2a2e0f73-7d70-46a7-9ec9-96c041f292dd" />
<img width="603" height="251" alt="p3-dhcp-scope-options" src="https://github.com/user-attachments/assets/a524a856-b6a9-4804-9d93-25194bebef07" />
<img width="809" height="199" alt="p3-dhcp-leases" src="https://github.com/user-attachments/assets/1645a91c-a71e-48fc-952f-0eb1151f761a" />
<img width="631" height="162" alt="p3-dhcp-failover" src="https://github.com/user-attachments/assets/b213494a-b18b-4f95-a06d-0e3015381776" />
<img width="1295" height="232" alt="p2-vm-switch-connections" src="https://github.com/user-attachments/assets/18aaf9bd-de72-4cb5-9fd3-a7bc243b3eae" />
<img width="1283" height="243" alt="p2-virtual-switch" src="https://github.com/user-attachments/assets/c70d74f0-5012-4c3c-b3fa-0b3983c26b90" />
<img width="1209" height="396" alt="p2-netnat" src="https://github.com/user-attachments/assets/2ccd579b-f7aa-4d9a-a847-a6663607e89b" />
<img width="1083" height="347" alt="p2-ad-ds-role" src="https://github.com/user-attachments/assets/34fcccaa-47c8-418b-ae13-296a229fc9a0" />
<img width="440" height="202" alt="p1-hyperv-folder-locations" src="https://github.com/user-attachments/assets/3888a628-d901-490d-acb6-195d3cfb5ff2" />
<img width="893" height="260" alt="p1-differencing-disks" src="https://github.com/user-attachments/assets/5b6bf836-8e5a-4af5-8909-d217a101d7b9" />
<img width="508" height="214" alt="p1-base-images" src="https://github.com/user-attachments/assets/81d338a1-9ee5-4e69-9ca8-5eec4d05ff72" />
<img width="1297" height="199" alt="p5-running-containers" src="https://github.com/user-attachments/assets/3576e066-e557-4dcd-b626-f0dfe8f69eaf" />
<img width="1262" height="122" alt="p5-rsiweb-bind-mount" src="https://github.com/user-attachments/assets/e480ce1c-f017-46f1-ac01-4a60a3ef04ac" />
<img width="627" height="112" alt="p5-port-bindings" src="https://github.com/user-attachments/assets/7a5b8547-bd37-4ea5-8e25-45eb4e15533d" />
<img width="1024" height="640" alt="p4-gpo-ws-updates" src="https://github.com/user-attachments/assets/51096fd5-262a-4209-976c-88542b5a2c40" />
<img width="990" height="585" alt="p4-gpo-ws-security" src="https://github.com/user-attachments/assets/6937f940-bbcf-4c5b-8f71-f4cb0a4f4239" />
<img width="1101" height="157" alt="p4-docker-images" src="https://github.com/user-attachments/assets/0f11d4ba-cfd3-45a2-9e39-0bd9f08f0b25" />
<img width="1293" height="226" alt="p4-container-templates" src="https://github.com/user-attachments/assets/eebff467-eab7-4008-88ae-9658c2d58c6a" />
<img width="829" height="269" alt="p4-bind-mount-folders" src="https://github.com/user-attachments/assets/b9ce9fde-6946-4b92-a862-e1a809e32ec7" />

## 💡 Skills Demonstrated
Windows Server administration · Hyper-V virtualization · Active Directory design · DHCP & DNS · Group Policy hardening · File share permissions · PowerShell verification · Docker containers · Technical documentation
