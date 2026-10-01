# 🛡️ Penetration Testing

![Tool](https://img.shields.io/badge/Tools-Nmap%20%7C%20Metasploit%20%7C%20Hydra%20%7C%20Wireshark%20%7C%20Nikto-red)
![Environment](https://img.shields.io/badge/Environment-Kali%20Linux%20%7C%20Metasploitable%202-informational)
![Category](https://img.shields.io/badge/Category-Penetration%20Testing-blue)
![Status](https://img.shields.io/badge/Status-Complete-success)

A full penetration test against a Metasploitable 2 virtual machine, covering the complete engagement lifecycle — lab setup, reconnaissance, scanning, vulnerability assessment, exploitation, password attacks, and network traffic analysis — carried out in an isolated Kali Linux lab.

---

## 📌 Project Overview

This project documents a controlled penetration test performed on a Metasploitable 2 virtual machine using Kali Linux, following the full engagement lifecycle from environment setup through exploitation and analysis. The goal was to identify and exploit real vulnerabilities in a deliberately vulnerable target, then translate the findings into a professional vulnerability report with actionable remediation steps.

## 🎯 Objectives

- Build an isolated virtual penetration testing lab
- Perform network reconnaissance and host discovery
- Scan for open ports, services, and versions
- Identify and assess vulnerabilities
- Ethically exploit a vulnerability and analyze impact
- Test password strength and analyze network traffic
- Report findings with actionable recommendations

## 🧰 Lab Setup

Two virtual machines were configured in Oracle VirtualBox on a host-only network adapter, isolating the test from external networks.

| Component | Configuration |
|---|---|
| Attacker Machine | Kali Linux |
| Target Machine | Metasploitable 2 |
| Network Mode | Host-only Adapter |
| Kali VM Specs | 4 GB RAM, 2 CPU cores, 40 GB storage |

![VirtualBox environment: attacker and target VMs running](./screenshots/fig1.0_virtualbox_environment.png)
*Figure 1.0 — VirtualBox environment: attacker and target VMs running*

![Kali Linux desktop (attacker machine)](./screenshots/fig1.1_kali_desktop.png)
*Figure 1.1 — Kali Linux desktop (attacker machine)*

![Metasploitable shell (target machine)](./screenshots/fig1.2_metasploitable_shell.png)
*Figure 1.2 — Metasploitable shell (target machine)*

> **Note:** the target's IP address changes across figures because the lab network's DHCP reassigned leases between sessions; the same Metasploitable target was used throughout.

## 🔎 Reconnaissance

Netdiscover and Nmap host discovery were used to identify live hosts on the lab network and confirm the target.

![netdiscover identifying connected hosts](./screenshots/fig2.0_netdiscover_hosts.png)
*Figure 2.0 — netdiscover identifying connected hosts*

![nmap -sn host discovery confirming the target is up](./screenshots/fig2.1_nmap_sn_host_discovery.png)
*Figure 2.1 — nmap -sn host discovery confirming the target is up*

## 🌐 Network Scanning and Service Enumeration

![Basic nmap scan: open ports and services](./screenshots/fig3.0_nmap_open_ports.png)
*Figure 3.0 — Basic nmap scan: open ports and services*

The scan returned an unusually large number of open services, including FTP, SSH, Telnet, HTTP, MySQL, and several remote-shell services (rlogin, rexec) — a strong indicator of a deliberately vulnerable host.

![nmap -sV service/version detection](./screenshots/fig3.1_nmap_sV_service_detection.png)
*Figure 3.1 — nmap -sV service/version detection*

Version detection flagged **vsftpd 2.3.4** and other outdated service versions on the target.

![nmap -O operating system detection](./screenshots/fig3.2_nmap_O_os_detection.png)
*Figure 3.2 — nmap -O operating system detection*

## 🧪 Vulnerability Assessment

![nmap --script vuln identifying the vsftpd 2.3.4 backdoor (CVE-2011-2523)](./screenshots/fig4.0_nmap_vuln_script_vsftpd_backdoor.png)
*Figure 4.0 — nmap --script vuln identifying the vsftpd 2.3.4 backdoor (CVE-2011-2523)*

## 🕸️ Web Application Security Testing

![Metasploitable web landing page exposing vulnerable applications](./screenshots/fig5.0_metasploitable_web_landing_page.png)
*Figure 5.0 — Metasploitable web landing page exposing vulnerable applications (DVWA, phpMyAdmin, TWiki, Mutillidae)*

![Nikto scan: outdated Apache/PHP and missing security headers](./screenshots/fig5.1_nikto_scan.png)
*Figure 5.1 — Nikto scan: outdated Apache/PHP and missing security headers*

## 💥 Exploitation Using Metasploit

The vsftpd 2.3.4 backdoor vulnerability was exploited using the `exploit/unix/ftp/vsftpd_234_backdoor` module.

![Configuring RHOST/LHOST for the exploit module](./screenshots/fig6.0_metasploit_configuring_exploit.png)
*Figure 6.0 — Configuring RHOST/LHOST for the exploit module*

![Successful exploitation: reverse shell / Meterpreter session opened](./screenshots/fig6.1_meterpreter_session_opened.png)
*Figure 6.1 — Successful exploitation: reverse shell / Meterpreter session opened*

**Impact:** the backdoor grants an unauthenticated root-level shell on the target, allowing full system compromise, data theft, or further lateral movement.

## 🔑 Password Security Testing

![Hydra brute-force against the FTP service](./screenshots/fig7.0_hydra_ftp_bruteforce.png)
*Figure 7.0 — Hydra brute-force against the FTP service*

Hydra recovered valid FTP credentials (`msfadmin:msfadmin`) from a small wordlist, confirming the target's exposure to weak/default password attacks.

## 📡 Network Traffic Analysis Using Wireshark

![Captured traffic between attacker and target](./screenshots/fig8.0_wireshark_captured_traffic.png)
*Figure 8.0 — Captured traffic between attacker and target*

![HTTP POST request to phpMyAdmin](./screenshots/fig8.1_wireshark_http_post_phpmyadmin.png)
*Figure 8.1 — HTTP POST request to phpMyAdmin*

![phpMyAdmin login credentials captured in cleartext via packet sniffing](./screenshots/fig8.2_wireshark_credentials_cleartext.png)
*Figure 8.2 — phpMyAdmin login credentials captured in cleartext via packet sniffing*

Because the web application traffic is unencrypted, credentials sent to phpMyAdmin were readable directly from the packet capture.

## 📊 Vulnerability Findings Summary

| Vulnerability | Affected Service / Port | Severity | Evidence |
|---|---|---|---|
| vsftpd 2.3.4 backdoor (CVE-2011-2523) | FTP – Port 21 | **Critical** | Nmap vuln script; Metasploit root shell |
| Weak / default credentials | FTP – msfadmin:msfadmin | High | Hydra brute-force |
| Unencrypted HTTP traffic exposing admin credentials | HTTP / phpMyAdmin – Port 80 | High | Wireshark packet capture |
| Outdated, unpatched web server stack | Apache 2.2.8, PHP 5.2.4 | Medium | Nikto scan |
| Missing HTTP security headers | HTTP – Port 80 | Medium | Nikto scan |
| Excessive unnecessary open services | Telnet, rlogin, rexec, NFS, etc. | Medium | Nmap port scan |

## ✅ Recommendations

| Recommendation | Addresses |
|---|---|
| Patch or replace vsftpd 2.3.4 with a current, non-backdoored release | Critical FTP backdoor |
| Enforce a strong password policy; disable default accounts | Weak credentials |
| Migrate web and admin interfaces to HTTPS/TLS | Cleartext credential exposure |
| Update Apache/PHP and apply vendor security headers | Outdated software, missing headers |
| Disable or remove unused services (Telnet, rlogin, rexec, NFS) | Excessive attack surface |
| Segment the network and restrict host-only lab access from production | General exposure reduction |

## 🧾 Conclusion

This engagement confirmed that the Metasploitable 2 target is highly vulnerable due to an outdated, unpatched service stack, weak credentials, and unencrypted traffic. The critical vsftpd backdoor alone enabled full root compromise. Applying the recommendations above — patching, credential hardening, encryption, and service minimization — would eliminate the most severe risks identified.

## 🛠️ Skills Demonstrated

- Penetration testing methodology (recon → scan → exploit → report)
- Network reconnaissance (Netdiscover, Nmap host discovery)
- Port/service/OS enumeration (Nmap)
- Vulnerability scanning (Nmap NSE, Nikto)
- Exploitation with Metasploit / Meterpreter
- Password attacks with Hydra
- Network traffic analysis with Wireshark
- Vulnerability reporting and remediation planning

## 📂 Repository Contents

```
Penetration-Testing-Metasploitable2/
│
├── CYB412_Penetration_Testing_Report.pdf   # Full lab report
├── screenshots/                            # All 17 figures referenced above
└── README.md                               # Project overview (this file)
```

---

## 👤 Author

**Nuhu Salihu**
Cybersecurity Student | Network Security Enthusiast | Aspiring Cybersecurity Analyst
📍 Kano, Nigeria
📧 salihunuhu994@gmail.com
🔗 GitHub: [@Nsalihu300](https://github.com/Nsalihu300)

---

⭐ If you find this project useful, feel free to explore the repository and the accompanying lab report.
