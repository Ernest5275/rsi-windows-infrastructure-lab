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

![Differencing disks](p1-differencing-disks.png)

## Phase 2: Networking & Active Directory Prerequisites
- Created internal virtual switch **VS-RSI-LAN** and a **NAT** network (10.2.2.0/24) on the host
- Connected all VMs to the switch and assigned static IPs to the servers
- Installed the AD DS role on all three servers and activated Windows Server

![NAT configuration](p2-netnat.png)

## Phase 3: Active Directory, DHCP, DNS & Shares
- Built a **parent/child forest**: `EF71RSI.local` and `GuardianDev.EF71RSI.local`
- Created OUs, security groups, and users for each department (Admin, HR, IT, Engineering, Operations) with a naming standard
- Joined both workstations to the domain in the **RSIWorkstations** OU
- Configured **DHCP with failover** (Hot Standby) between two servers, with scope options for router, DNS, and domain name
- Created departmental **file shares** with group-based share and NTFS permissions
- Added a DNS **reverse lookup zone** (2.2.10.in-addr.arpa)

![Parent and child domains](p3-parent-child-domains.png)
![DHCP failover](p3-dhcp-failover.png)
![RSI shares and permissions](p3-rsi-shares-permissions.png)

## Phase 4: Group Policy & Docker
- **RSI-WS-Security GPO** (linked and enforced on RSIWorkstations): disables the built-in Administrator and Guest accounts, renames both, and uses Restricted Groups to control Remote Desktop Users
- **RSI-WS-Updates GPO**: automatic updates every Friday at 01:00 from an internal update server
- Installed Docker, pulled Windows Server Core, and built two reusable images: **web-image-template** (with the IIS Web Server role) and **std-image-template**
- Created host folders for container bind mounts

![Security GPO settings](p4-gpo-ws-security.png)
![Docker images](p4-docker-images.png)

## Phase 5: Finalizing Docker Containers
- Deployed **RSIWeb** from the web template, publishing container port 80 on host port 5678
- Deployed **RSIStd** from the standard template
- Created a separate Docker NAT network, **RSI-Net** (192.168.1.0/24), for the standard server
- Attached persistent **bind mounts**: `D:\RSIData1 → C:\webdata` (RSIWeb) and `D:\RSIData2 → C:\stddata` (RSIStd)

![Running containers](p5-running-containers.png)
![Bind mount](p5-rsiweb-bind-mount.png)

## 📸 All Screenshots
Every verification screenshot is in this repository, named by phase (p1 through p5).
<img width="631" height="162" alt="p3-dhcp-failover" src="https://github.com/user-attachments/assets/0b38b035-f94e-4cdf-a71c-3344fd02e16f" />
<img width="1295" height="232" alt="p2-vm-switch-connections" src="https://github.com/user-attachments/assets/65958692-2eae-4668-93d9-9e21181d92cf" />
<img width="1283" height="243" alt="p2-virtual-switch" src="https://github.com/user-attachments/assets/697ce102-59af-491b-9f34-94779d00fa49" />
<img width="1209" height="396" alt="p2-netnat" src="https://github.com/user-attachments/assets/39c6a2ae-ac3f-4360-b271-763c264ab88a" />
<img width="1083" height="347" alt="p2-ad-ds-role" src="https://github.com/user-attachments/assets/c5846659-09c0-4923-a0dd-cb582026295b" />
<img width="440" height="202" alt="p1-hyperv-folder-locations" src="https://github.com/user-attachments/assets/62b34142-83b8-4df5-a83b-a5828c328d1a" />
<img width="893" height="260" alt="p1-differencing-disks" src="https://github.com/user-attachments/assets/b623842c-0ee9-44ab-9fe3-11c47dc4d6bf" />
<img width="508" height="214" alt="p1-base-images" src="https://github.com/user-attachments/assets/5b7b3936-1bad-401b-9150-ed568c1f05a8" />
<img width="1297" height="199" alt="p5-running-containers" src="https://github.com/user-attachments/assets/0008a425-0e4b-4eb0-8181-8595a6c62c6c" />
<img width="1262" height="122" alt="p5-rsiweb-bind-mount" src="https://github.com/user-attachments/assets/73ee34c1-d37d-4d4d-9d7b-f50ea00f07e5" />
<img width="627" height="112" alt="p5-port-bindings" src="https://github.com/user-attachments/assets/15e2df9a-47d4-49f8-9924-ad92aa7bd7fa" />
<img width="1024" height="640" alt="p4-gpo-ws-updates" src="https://github.com/user-attachments/assets/b7e61d66-586f-4075-aedd-5cf2595b983d" />
<img width="990" height="585" alt="p4-gpo-ws-security" src="https://github.com/user-attachments/assets/6a9679ce-5319-4144-b017-710935c06a9f" />
<img width="1101" height="157" alt="p4-docker-images" src="https://github.com/user-attachments/assets/37cdbb7f-0bf7-4e09-b11c-8e454b90cf3e" />
<img width="1293" height="226" alt="p4-container-templates" src="https://github.com/user-attachments/assets/636bf766-a8d4-4004-ac50-c568156c2851" />
<img width="829" height="269" alt="p4-bind-mount-folders" src="https://github.com/user-attachments/assets/c86fbc48-0168-4e4e-b8ff-caa1739a8a5e" />
<img width="1286" height="674" alt="p3-rsi-shares-permissions" src="https://github.com/user-attachments/assets/10867901-0686-4345-ac16-6b1abc1f4dca" />
<img width="624" height="198" alt="p3-rsi-ous" src="https://github.com/user-attachments/assets/090e73bc-8b1b-433c-bc94-74e3c3c63305" />
<img width="1284" height="528" alt="p3-rsi-groups" src="https://github.com/user-attachments/assets/a0d964e7-75f0-4a91-8bef-e08712efacc0" />
<img width="1095" height="253" alt="p3-parent-child-domains" src="https://github.com/user-attachments/assets/87bb8b54-6c95-4641-973c-e9a9eb5a69e6" />
<img width="1030" height="145" alt="p3-dns-reverse-lookup" src="https://github.com/user-attachments/assets/794709a9-63be-413b-a945-4931696eb80b" />
<img width="603" height="251" alt="p3-dhcp-scope-options" src="https://github.com/user-attachments/assets/ce466235-9c8b-4242-90d7-13847ee266bd" />
<img width="809" height="199" alt="p3-dhcp-leases" src="https://github.com/user-attachments/assets/f024084a-be21-4671-a567-3e8104513973" />

## 💡 Skills Demonstrated
Windows Server administration · Hyper-V virtualization · Active Directory design · DHCP & DNS · Group Policy hardening · File share permissions · PowerShell verification · Docker containers · Technical documentation
