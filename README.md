# Task 1: Scan Your Local Network for Open Ports

## Objective
Discover open ports on devices in my local network using Nmap, and understand the network exposure they create.

## Tools Used
- Nmap 7.991 (Windows)
- Wireshark (optional, used to inspect SYN packets)
- PowerShell 7.6.6 (run as Administrator)

> Only my own machine/network was scanned.

## What I Did
1. Installed Nmap from the official website (nmap.org).
2. Found my network details with `ipconfig`:

   | Adapter | IPv4 Address | Subnet Mask | Range |
   |---------|--------------|-------------|-------|
   | Ethernet 6 (host-only virtual adapter) | 192.168.56.1 | 255.255.255.0 | 192.168.56.0/24 |
   | Wi-Fi 2 (real network) | 192.168.1.5 | 255.255.255.0 | 192.168.1.0/24 |

3. Ran a TCP SYN scan on the 192.168.56.0/24 range and saved the output:
   ```
   nmap -sS 192.168.56.0/24 -oN scan_results.txt -oX scan_results.xml
   ```
4. Noted the open ports, researched the services, and identified the risks.
5. Captured traffic in Wireshark on the Wi-Fi interface and filtered for SYN packets.

## Scan Summary
- **Addresses scanned:** 256 (192.168.56.0 to 192.168.56.255)
- **Hosts up:** 1
- **Scan time:** 11.84 seconds
- **Closed ports on the live host:** 997 (responded with reset)

## Results

**Host: 192.168.56.1** (my own Windows PC, host-only adapter)

| IP Address | Port | State | Service | Description |
|------------|------|-------|---------|-------------|
| 192.168.56.1 | 135/tcp | open | msrpc | Microsoft RPC endpoint mapper |
| 192.168.56.1 | 139/tcp | open | netbios-ssn | NetBIOS Session Service (legacy file/printer sharing) |
| 192.168.56.1 | 445/tcp | open | microsoft-ds | SMB (Windows file sharing) |

Full output: [`scan_results.txt`](scan_results.txt) and [`scan_results.xml`](scan_results.xml)

![Nmap scan](screenshots/nmap_scan.png)

## Services and Security Risks

| Port | Service | What it does | Potential risks |
|------|---------|--------------|-----------------|
| 135 | MSRPC | Lets Windows clients find RPC services on the machine | Reveals information about running services; historically exploited by worms (e.g. Blaster); should never be exposed to the internet |
| 139 | NetBIOS-SSN | Legacy file and printer sharing over NetBIOS | Information disclosure (names, shares), credential capture/relay attacks, outdated protocol |
| 445 | SMB (microsoft-ds) | Modern Windows file and printer sharing | Exploited by EternalBlue/WannaCry (MS17-010), brute-force and credential relay attacks, ransomware spread inside a network |

## Findings
- The scan found only one live host, my own PC on the host-only adapter. The three open ports (135, 139, 445) are the standard Windows file-sharing and RPC services.
- These ports are low risk on a private, isolated network like this, but they would be a real exposure on a public or untrusted network (café Wi-Fi, college network), because SMB and RPC have a long history of serious vulnerabilities.
- The other 997 scanned ports were closed, so the exposure is limited to Windows networking services.

## How to Reduce the Exposure
- Keep Windows fully updated (patches fix the known SMB/RPC vulnerabilities).
- Set untrusted networks to the **Public** profile so Windows Firewall blocks inbound file sharing.
- Disable SMBv1 and turn off NetBIOS over TCP/IP if it isn't needed.
- Use Windows Defender Firewall rules to block inbound 135, 139 and 445 from other devices.
- Never forward these ports on the router to the internet.

## Wireshark Analysis (Optional)
I captured traffic on the **Wi-Fi 2** interface and applied the filter:
```
tcp.flags.syn==1 && tcp.flags.ack==0
```
This shows only the first packet of each TCP handshake (SYN without ACK), which is the same kind of packet a SYN scan sends. 3 of 806 captured packets matched. Each shows my machine (192.168.1.5) sending a SYN to port 443 of an external server, with options such as MSS=1460, window scaling and SACK permitted.

![Wireshark SYN filter](screenshots/wireshark_syn_filter.png)

Note: the Nmap scan targeted the 192.168.56.0/24 host-only adapter, so this capture shows normal HTTPS connection attempts on the Wi-Fi interface rather than the scan packets themselves. It demonstrates what SYN packets look like on the wire.


## Conclusion
This task showed how a simple SYN scan can quickly reveal which services a machine exposes. My PC exposes the standard Windows networking ports 135, 139 and 445, which are fine on a private network but risky on untrusted ones. Keeping systems patched, using a firewall, and closing unneeded services are the main ways to reduce this exposure.

## Repository Structure
```
├── README.md
├── scan_results.txt
├── scan_results.xml
└── screenshots/
    ├── nmap_scan.png
    └── wireshark_syn_filter.png
```
