# T-Pot Honeynet — Weekly Threat Intelligence Project

Internet-facing honeynet for live threat intelligence collection and analysis. Deployed May 29, 2026.

## About This Project

This project is a live internet-facing honeynet that collects real-world attack data 24/7 from threat actors around the globe. Every week, the top 3 most active or most interesting attacking IPs are selected from the live data, profiled using OSINT tools, and published as structured weekly threat intelligence reports.

The goal of this project is to build hands-on experience in Threat Intelligence Analysis and Threat Hunting. Rather than working with simulated or lab-generated data, every finding in this project comes from real attackers hitting a live system. Over time, new campaigns are identified, new threat actors are profiled, and the reports evolve week over week to capture trending attack patterns and emerging threats.

## Lab Architecture

![T-Pot Honeynet Lab Architecture](Screenshots/T-Pot%20Honeynet%20Lab%20Architecture.png)

The honeynet runs on an HP EliteDesk 800 G6 Mini with Proxmox VE installed as the hypervisor. On top of Proxmox sits a Debian 13 virtual machine where T-Pot lives. T-Pot runs 20+ honeypot services simultaneously, each as its own isolated Docker container. All inbound internet traffic flows through an AT&T Fiber gateway configured in IP Passthrough mode into a TP-Link Archer BE3500 lab router. The router has a DMZ rule that forwards all incoming traffic directly to the T-Pot VM at 192.168.0.221. The lab network (192.168.0.0/24) is completely separate from the home network with no connection between them.

## How It Works

When an attacker on the internet scans or probes the public IP, their traffic passes through the AT&T gateway, hits the lab router, and gets forwarded to the T-Pot VM. T-Pot then routes each connection to the right honeypot based on which port the attacker is targeting. The attacker thinks they found a real vulnerable system. Every credential they try, every command they run, and every payload they send gets logged, processed, and stored in Elasticsearch where it can be searched and visualized in Kibana.

## How I Am Protected

Pointing a home machine at the open internet sounds risky. The reason it is safe comes down to a layered defense strategy where each layer prevents attackers from going further even if the one before it is breached.

**Layer 1 — Network Segmentation**

The honeynet runs on its own dedicated network (192.168.0.0/24) behind the lab router. The home network runs on a completely separate subnet behind the AT&T gateway. There is no routing between the two networks. Even if an attacker fully compromised the T-Pot VM, they would be stuck on the lab network with no path to any home devices.

**Layer 2 — Docker Container Isolation**

Every honeypot service runs inside its own isolated Docker container. If an attacker broke out of a honeypot service, they would land inside a container with no access to other containers or the host operating system. Getting further would require a Docker container escape, which is a significantly harder attack.

**Layer 3 — Proxmox VM Isolation**

T-Pot runs as a virtual machine on Proxmox. Even if an attacker achieved a container escape, they would only be inside the Debian VM and not on the physical machine. Getting to the hardware would then require a full hypervisor escape on top of that.

**Layer 4 — No Personal Devices on Lab Network**

No personal computers, phones, or other devices are connected to the lab router. The only device on the lab network is the G6 Mini running the honeynet. There is nothing else to reach even if an attacker made it onto the lab network.

**Layer 5 — Management Access Restricted**

The T-Pot web interface, SSH, and Kibana are never exposed to the public internet. These management ports are only reachable from the password-protected lab WiFi network or through Tailscale VPN. An attacker on the internet has no way to reach the management layer.

The combined effect is that even in the worst case scenario where every layer is fully compromised, an attacker would still be isolated inside a VM on a network with nothing else on it.

## T-Pot Honeynet Services

T-Pot is an open source honeynet platform developed by Telekom Security that bundles over 20 individual honeypot services into a single deployment. Each service runs as its own Docker container and emulates a different type of vulnerable system or network service. When an attacker connects to a port, T-Pot automatically routes them to the honeypot that matches what they are targeting. The attacker believes they found a real system while every action they take is silently captured and logged.

The table below covers the 10 honeypot services that are actively receiving attacker traffic on this honeynet. While T-Pot runs more services than what is listed here, these are the ones that real attackers are hitting based on live data from the honeynet.

| Honeypot | Port(s) | Protocol | Attack Type | What It Captures |
|---|---|---|---|---|
| **Cowrie** | 22, 23 | SSH / Telnet | Brute force, post-exploitation | Emulates a Linux server. Accepts logins and puts attackers into a fake shell. Captures every command typed, credential tried, and malware download attempted. |
| **Dionaea** | 3306, 445, 80 | MySQL / SMB / HTTP | Database ransomware, malware | Catches bots that look for exposed databases, authenticate with default credentials, wipe the database, and leave a ransom note demanding cryptocurrency. |
| **Sentrypeer** | 5060 | SIP / VoIP | VoIP toll fraud | Emulates a VoIP server. Catches attackers who hijack phone systems to make fraudulent international calls billed to the victim. |
| **Honeytrap** | All ports | TCP / UDP | Port scanning, reconnaissance | Listens on thousands of ports at once. Catches everything that does not match a more specific honeypot including scans, probes, and raw exploit attempts. |
| **Heralding** | 21, 3389, 5900, 25 | FTP / RDP / VNC / SMTP | Credential stuffing | Emulates multiple login services. Logs every username and password combination that automated bots try across FTP, Remote Desktop, VNC, and email. |
| **Tanner** | 80, 443 | HTTP / HTTPS | Web application attacks | Simulates a vulnerable website. Captures SQL injection, file inclusion, remote code execution attempts, and automated vulnerability scanners. |
| **Ciscoasa** | 443 | HTTPS | Cisco firewall exploitation | Emulates a Cisco ASA firewall. Catches exploit attempts targeting known Cisco vulnerabilities. |
| **Adbhoney** | 5555 | ADB | Android device attacks | Emulates an Android device with the debug port exposed. Catches attackers trying to install malware or crypto miners on Android devices. |
| **Redishoneypot** | 6379 | Redis | Database attacks | Emulates an exposed Redis database. Catches attackers trying to abuse misconfigured Redis instances to gain access to a server. |
| **ElasticPot** | 9200 | HTTP | Elasticsearch exploitation | Emulates an exposed Elasticsearch database. Catches data theft attempts and ransom note injections. |

## Problems Encountered and How I Fixed Them

**T-Pot containers not starting after reboot**

The T-Pot systemd service was never registered because the install script failed partway through due to a network timeout. The fix was manually creating a systemd service file and enabling it so T-Pot starts automatically on every boot.

**DNS breaking inside the VM after reboot**

Tailscale kept overwriting the DNS configuration file with an IPv6 DNS server that Docker could not use, causing container image pulls to fail with DNS resolution errors. The fix was manually setting Google DNS (8.8.8.8) and locking the file so nothing could overwrite it on reboot.

**T-Pot web interface login not working**

The web credentials entry in T-Pot's config file was either empty or had a malformed encoded string, causing the initialization container to abort on every startup. The fix was regenerating the credentials correctly by stripping the trailing newline before encoding, then writing the value into the config file using Python to avoid character escaping issues.

**No internet traffic reaching the honeynet**

After setting up the lab router DMZ, the honeynet was receiving no traffic at all. The AT&T Fiber gateway was intercepting all inbound connections before they could reach the lab router. The fix was enabling IP Passthrough mode on the AT&T gateway so all inbound traffic passes directly through to the lab router.

**VM getting a different IP after every reboot**

The T-Pot VM was using DHCP so it received a different IP address each time it restarted, breaking the router DMZ rule. The fix was configuring a static IP directly in the VM's network interface configuration file.

**Proxmox firewall blocking VM traffic**

The T-Pot VM could not communicate with the lab router even though the Proxmox host could. The Proxmox built-in firewall was enabled on the VM's network interface and silently dropping traffic. The fix was disabling the firewall on the VM's network device in the Proxmox hardware settings.

## Weekly Threat Intelligence Reports

This is an ongoing project to build real experience in Threat Hunting and Threat Intelligence Analysis. Each week, attack data from the live honeynet is reviewed and the most interesting IPs are selected for deeper investigation. This is not limited to just the highest volume attackers. Sometimes an IP with lower hit counts is selected because the campaign type, the infrastructure behind it, or the OSINT findings make it more interesting from a threat intelligence perspective.

Each report covers threat actor profiling, OSINT findings from Spiderfoot, Kibana attack dashboards, MITRE ATT&CK mapping, indicators of compromise, and analyst recommendations. Over time these reports will show how attack patterns evolve and what campaigns are active against internet-facing systems.

All weekly reports are in the [weekly-reports](weekly-reports/) folder.

### Report Index

| Week | Period | Report |
|---|---|---|
| Week 1 | May 29 – May 31, 2026 | [WTIR-01-2026](weekly-reports/WTIR-01-2026.md) |

## Lab Environment

| Component | Details |
|---|---|
| Platform | T-Pot 24.04.1 HIVE |
| Hardware | HP EliteDesk 800 G6 Mini |
| Hypervisor | Proxmox VE |
| VM OS | Debian 13 |
| Lab Router | TP-Link Archer BE3500 |
| ISP | AT&T Fiber (IP Passthrough) |
| Remote Access | Tailscale VPN |
| Deployment Date | May 29, 2026 |

## Author

**Michael Mensah**

[![GitHub](https://img.shields.io/badge/GitHub-KingMike77-black?logo=github)](https://github.com/KingMike77)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Michael%20Mensah-blue?logo=linkedin)](https://linkedin.com/in/michael-mensah-cybersecurity)
