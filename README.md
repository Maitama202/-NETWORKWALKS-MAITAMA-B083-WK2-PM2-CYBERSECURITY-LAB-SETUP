# Networkwalks Cybersecurity Internship — Week 2

## Footprinting, OSINT & Network Scanning

This repository contains my Week 2 practical cybersecurity work completed in a Kali Linux and Windows/Zenmap lab environment.

The practical work documented here covers:

- **W2-PM1 — Footprinting & Reconnaissance with Multiple Kali Tools**
- **W2-PM4 — Footprinting & Reconnaissance with theHarvester**
- **W2-PM5 — Network Scanning with Zenmap/Nmap**

> **Scope and authorization:** Security testing should only be performed against systems that are owned/controlled by the tester or covered by current written authorization. The sample authorization letter supplied with the course material is dated 17–24 August 2026 and names a different tester, so it is not treated here as current authorization for this report. No exploitation, brute force, privilege escalation, denial-of-service, or data modification was performed.

---

## 1. Objectives

The Week 2 practicals were designed to develop practical skills in:

1. Domain and web reconnaissance.
2. DNS and HTTP information gathering.
3. Web Application Firewall identification.
4. Open-source intelligence (OSINT) collection.
5. Local network host discovery.
6. Evidence collection and professional security reporting.

---

## 2. Tools Used

| Tool | Purpose |
|---|---|
| WHOIS | Domain registration and name-server information |
| WhatWeb | Web technology fingerprinting |
| Nslookup | DNS-to-IP resolution |
| curl | HTTP response-header inspection |
| wafw00f | WAF fingerprinting |
| DNSRecon | DNS record enumeration |
| theHarvester | OSINT collection of emails, hosts and infrastructure data |
| Zenmap / Nmap | Local network host discovery and topology visualization |

---

## 3. W2-PM1 — Footprinting & Reconnaissance

### Target

`networkwalks.com`

### 3.1 WHOIS

Command:

```bash
whois networkwalks.com
```

Key observations:

- Registrar: **GoDaddy.com, LLC**
- Creation date: **2019-11-06**
- Registry expiration date: **2027-11-06**
- Name servers included:
  - `NS6135.HOSTGATOR.COM`
  - `NS6136.HOSTGATOR.COM`
- DNSSEC was reported as **unsigned**.
- Registrant information was privacy-protected.

### 3.2 WhatWeb

Command:

```bash
whatweb networkwalks.com
```

Observed information included:

- Apache web server
- IP address: `192.232.216.135`
- WordPress **7.1.1**
- WordPress Download Manager **3.3.58**
- jQuery **3.7.1**
- Bootstrap **7.1.1**
- Google Tag Manager
- Site title: **Networkwalks Academy**
- HTTPS returned `200 OK`; HTTP redirected with `301 Moved Permanently`.

These are fingerprinting observations only and do not, by themselves, establish a vulnerability.

### 3.3 Nslookup

Command:

```bash
nslookup networkwalks.com
```

Observed result:

```text
networkwalks.com
192.232.216.135
```

The domain resolved to `192.232.216.135`.

### 3.4 HTTP Headers with curl

Command:

```bash
curl -I https://networkwalks.com
```

Observed:

- HTTP status: `200`
- Server: `Apache`
- Content type: `text/html; charset=UTF-8`
- WordPress-related cache/header information
- WordPress REST API link: `/wp-json/`
- Secure/HttpOnly cookie attributes were present for the observed WordPress Download Manager cookie.

### 3.5 WAFW00F

Command:

```bash
wafw00f networkwalks.com
```

Observed result:

```text
The site https://networkwalks.com is behind ModSecurity (SpiderLabs) WAF.
```

The tool reported two requests for the detection.

### 3.6 DNSRecon

Command:

```bash
dnsrecon -d networkwalks.com
```

Observed records included:

- SOA record
- Name servers
- MX record
- A record
- TXT/SPF records
- SRV/autodiscover records
- DNSSEC query returned no answer
- DNS software information was reported as BIND `9.16.23-RH`
- 8 SRV records were reported in the captured run.

---

## 4. W2-PM4 — theHarvester

### 4.1 Help / Tool Verification

Command:

```bash
theHarvester -h
```

The installed version shown in the evidence was **theHarvester 4.11.1**.

### 4.2 Baidu Search

Command:

```bash
theHarvester -d microsoft.com -l 1000 -b baidu
```

Observed:

- Target: `microsoft.com`
- Emails found: **3**
- IPs found: **0**
- Hosts found: **7**
- People found: **0**

The three returned email strings were captured in the evidence file. Because this repository is public-facing, the README intentionally does not reproduce every harvested third-party email address.

### 4.3 All Sources

Command:

```bash
theHarvester -d microsoft.com -l 50 -b all
```

The run produced:

- **50 IPs**
- **3 emails**
- **9,965 hosts**
- **6 ASNs**
- **3 interesting URLs**
- No LinkedIn users in the captured result.

The run also generated numerous API-key warnings and source-specific failures because many providers require API credentials. Therefore, the `-b all` result should be interpreted as a collection from the sources that were available rather than as an exhaustive representation of Microsoft's infrastructure.

The large host count also contains wildcard-style and other tool-generated entries. These should be validated before being treated as confirmed assets.

---

## 5. W2-PM5 — Zenmap / Nmap

### Scan target

```text
192.168.215.0/24
```

### Scan command

```bash
nmap -sn 192.168.215.0/24
```

### Observed result

The captured Nmap output reported:

- **256 IP addresses scanned**
- **2 hosts up**
- `192.168.215.1`
- `192.168.215.247`

For `192.168.215.247`, the evidence displayed a MAC address beginning:

```text
D8:93:D4:51:D3:1E
```

with vendor identification shown as **Xiaomi Communications**.

The Zenmap topology view also displayed `192.168.56.1`. This address is outside the scanned `192.168.215.0/24` range and appears in the topology as a separate local/virtual interface, so it was **not counted as one of the two hosts discovered by the stated scan**.

---

## 6. Key Findings

| Area | Observation | Security meaning |
|---|---|---|
| Domain registration | Registration and DNS information is publicly visible | Useful for infrastructure profiling |
| Web fingerprinting | Apache, WordPress 7.1.1 and WP Download Manager 3.3.58 detected | Software/version information can support further defensive review |
| DNS | A, MX, TXT/SPF, NS and SRV records were identified | Provides an external view of DNS infrastructure |
| HTTP | Server and WordPress-related headers were exposed | Adds to technology fingerprinting |
| WAF | ModSecurity (SpiderLabs) detected | Confirms presence of a defensive layer; not a vulnerability |
| OSINT | theHarvester returned emails, hosts, IPs and ASNs | Demonstrates how much infrastructure can be discovered from public sources |
| Local network | Two hosts responded in `192.168.215.0/24` | Useful for asset inventory and identifying unexpected devices |

**Important:** These are reconnaissance and discovery observations, not confirmed vulnerabilities.

---

## 7. Recommendations

1. Review publicly exposed software/version information and keep web components updated.
2. Periodically review HTTP headers for unnecessary information disclosure.
3. Review DNS, MX, TXT/SPF and SRV records and remove obsolete entries.
4. Continue monitoring and correctly configuring the WAF.
5. Maintain an accurate inventory of devices on the local network.
6. Investigate any device that is not recognized or authorized.
7. Validate OSINT findings before treating harvested hosts or emails as confirmed assets.
8. Configure required API integrations when a complete theHarvester assessment is necessary.
9. Keep screenshots and raw command output as evidence for reproducibility.
10. Perform all future reconnaissance and scanning only within a clearly documented authorization scope.

---

## 8. Limitations

- The `theHarvester -b all` run produced many missing-API-key warnings, so not every supported source returned data.
- OSINT results are time-dependent and can change as search engines and public data sources change.
- A detected software version is not proof that the software is vulnerable.
- A live host discovered by Nmap is not automatically a security weakness.
- The evidence supplied for this report did not include execution results for W2-PM2 (GHDB) or W2-PM3 (Maltego), so those modules are not claimed as completed in this README.

---

## 9. Evidence

The accompanying Week 2 report contains the screenshots collected during the practical work, including:

- WHOIS
- WhatWeb
- Nslookup
- curl
- WAFW00F
- DNSRecon
- theHarvester help
- theHarvester Baidu search
- theHarvester all-source search
- Zenmap/Nmap host discovery
- Zenmap topology

---

## 10. Conclusion

Week 2 provided practical experience in reconnaissance, OSINT and network discovery. The exercises demonstrated that a security professional can collect substantial information about a domain and a local network without performing exploitation.

The main lesson from the practical was that reconnaissance findings must be interpreted carefully. Publicly visible technology, DNS records, IP addresses, harvested hosts and live network devices are useful observations, but they do not automatically represent vulnerabilities.

Professional cybersecurity work therefore combines technical collection with validation, evidence preservation, risk interpretation and strict authorization boundaries.

---

## Author

**Cybersecurity Internship — Networkwalks | Week 2**

Repository contents:

- `README.md`
- `Week2_Penetration_Testing_Report.docx`
- Evidence screenshots from the practical exercises
