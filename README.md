# NETWORKWALKS-OMER-B084-WK4-PM-MEDIROZA-PENETRATION-TESTING

<div align="center">
<h3>PENETRATION TESTING REPORT</h3>
<h3>MEDIROZA GENERAL HOSPITAL</h3>
<p>W4-PM-FINAL | CYBERSECURITY | NETWORKWALKS</p>
</div>

| Field | Details |
| :--- | :--- |
| Pentester Name | Omer Khalid Omer Ali Bawazir |
| Program/Batch | B084-Networkwalks |
| Date | 25 November 2026 |
| Modules completed | W4-PM (Milestones 1, 2, 3 & 4) |
| Client/Target | Mediroza General Hospital (https://medirozahospital.com) |
| Permission secured from client? | Yes (Written authorization granted by Networkwalks) |
| Scope | Full Black-box Penetration Test |

---

## 1. Liability Disclaimer

I have performed these activities only on systems and devices where I had secured written permission or that I own myself. All materials are for education and research purposes only. Do not use anything from here to break the law. The instructor, the authors, and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job, and a permanent record. All activities were performed within the authorized scope of the assigned educational lab.

---

## 2. Introduction

This report documents the Week 4 Penetration Testing Project against **Mediroza General Hospital**. The objective was to conduct a full black-box penetration test to identify vulnerabilities, exploit them to demonstrate real impact, and document all findings. The assessment successfully demonstrated initial access via SQL Injection, data extraction through password cracking, and the discovery of critical data exposure (employee salaries and shareholder details) through an exposed legacy directory and database backup.

---

## 3. Tools Used

| Tool | Purpose |
| :--- | :--- |
| **Kali Linux & Windows** | Operating systems used for testing |
| **Nikto** | Web server scanning & directory enumeration |
| **ffuf** | Fuzzing and directory brute-forcing |
| **WhatWeb** | Web technology fingerprinting |
| **Nslookup / DNSRecon** | DNS enumeration and IP resolution |
| **Curl** | HTTP header inspection |
| **Wafw00f** | Web Application Firewall detection |
| **Mozilla Firefox** | Manual browsing & SQL injection testing |
| **NetworkWalks Hash Calculator** | Extracting crackable `$pdf$` hashes from encrypted PDFs |
| **NetworkWalks Password Cracker** | Dictionary attack to recover PDF passwords |

---

## 4. Activities Performed

### 4.1 Reconnaissance & Footprinting (Pre-Attack Phase)

Before any exploitation, extensive reconnaissance was conducted to map the target's infrastructure, identify technologies, and discover hidden entry points.

#### 4.1.1 Web Technology Fingerprinting (WhatWeb)
**Command:**
```bash
whatweb medirozahospital.com
```

**Output Summary:**

- IP Address: `199.188.201.16`
- Web Server: `LiteSpeed`
- HTTP Status: `403 Forbidden` (on root and HTTPS)
- Headers: `x-turbo-charged-by`

**Finding:** The target uses the LiteSpeed web server. The root directory returns a 403 Forbidden, indicating that directory listing is disabled at the root level, but the IP address is exposed for further scanning.


#### 4.1.2 DNS Resolution (Nslookup & DNSRecon)

```bash
nslookup medirozahospital.com
dnsrecon -d medirozahospital.com
```

**Output Summary:**
- A Record: `199.188.201.16`
- Name Servers (NS): `dns1.namecheaphosting.com`, `dns2.namecheaphosting.com`
- MX Records: `mx1/mx2/mx3-hosting.jellyfish.systems`
- TXT (SPF): `v=spf1 +a +mx +ip4:199.188.201.7 +ip4:199.188.201.38 include:spf.web-hosting.com ~all`
- TXT (DMARC): `v=DMARC1; p=none;`
- SRV Records: `_autodiscover._tcp.medirozahospital.com` pointing to `cpanelemaildiscovery.cpanel.net` (Port 443).

**Finding:** The domain is hosted on Namecheap. The presence of cPanel autodiscover SRV records confirms the backend is a **cPanel server**. The DMARC policy is set to `p=none`, meaning the domain is not strictly enforcing email authentication (vulnerable to email spoofing).


#### 4.1.3 HTTP Header Inspection (Curl)

```bash
curl -I https://medirozahospital.com
```

**Output Summary:**
- `HTTP/2 200`
- Server: `openresty/1.31.1.1`
- Security Headers Present:
   - `x-content-type-options: nosniff`
   - `referrer-policy: strict-origin-when-cross-origin`
   - `x-frame-options: SAMEORIGIN`
   - `content-security-policy: script-src 'nonce-...'`

**Finding:** While WhatWeb identified LiteSpeed, Curl reveals the server is running **OpenResty** (a modified Nginx). This suggests a reverse proxy is in place. The presence of CSP and X-Frame-Options indicates some level of security hardening on the frontend.


#### 4.1.4 Web Application Firewall Detection (Wafw00f)

```bash
wafw00f medirozahospital.com
```

**Output Summary:**
- `[-] No WAF detected by the generic detection`

**Finding:** No Web Application Firewall (WAF) is blocking requests. This is a critical finding, as it means our subsequent SQL Injection and directory brute-forcing attacks will not be easily blocked.


#### 4.1.5 Directory Enumeration (Robots.txt & Nikto & ffuf)

```bash
curl https://medirozahospital.com/robots.txt

nikto -h https://medirozahospital.com/ -o nikto.txt

ffuf -u https://medirozahospital.com/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

**Output Summary:**
- **Robots.txt** revealed hidden directories:
```text
Disallow: /patient/
Disallow: /staff/
Disallow: /old/
```

- **Nikto** flagged directory indexing enabled on `/staff/`, `/patient/`, and `/old/`.

- **ffuf** confirmed the existence of these directories and returned `200 OK` with directory listings.

**Finding:** The `robots.txt` file acts as a roadmap for attackers, revealing sensitive directories. Directory indexing is enabled on these paths, allowing us to browse files directly (which will be crucial for M3).


### 4.2 Milestone 1: Initial Access (M1)




### 4.3 Milestone 2: Data Extraction (M2)



### 4.4 Milestone 3: Critical Data Exposure (M3)



## 5. Risk Analysis / Impact



## 6. Recommendations

1. **Fix SQL Injection:** Use Parameterized Queries (Prepared Statements) for all database interactions.

2. **Remove Exposed Files:** Delete the `/old/` directory and the `mediroza_db_backup_2019.sql` file from the production web server immediately.

3. **Disable Directory Indexing:** Configure the LiteSpeed/OpenResty web server to disable directory listing (`Options -Indexes`).

4. **Enforce Strong Passwords:** Implement a password policy for PDFs and user accounts that requires length, complexity, and special characters.

5. **Secure Error Messages:** Return generic error messages for both incorrect usernames and passwords.

6. **Enforce DMARC:** Change the DMARC policy from `p=none` to `p=reject` or `p=quarantine` to prevent email spoofing.

7. **Add 2FA:** Implement Multi-Factor Authentication for the Patient and Staff portals.


## 7. Conclusion

During Week 4 of my Cybersecurity & Ethical Hacking internship, I conducted a full black-box penetration test against Mediroza General Hospital. I successfully exploited a SQL injection vulnerability to bypass authentication, extracted patient data, cracked weak PDF encryptions, and discovered critical data exposures (salaries and shareholder details) via an exposed legacy directory and database backup. This project reinforced the importance of secure coding practices, proper server configurations, and the dangers of leaving sensitive files exposed on production servers. All activities were conducted within the authorized scope of the educational lab.


## 8. Evidences Collected

### Reconnaissance

* **1- Whatweb:**
  <img src="screenshots/Recon/1-whatweb.png" alt="1- Whatweb">

* **2- DNS-Resolution (dnsrecon)**
  <img src="screenshots/Recon/2-DNS-Resolution.png" alt="2- DNS Resolution">

* **3- curl-headers:**
  <img src="screenshots/Recon/3-curl-headers.png" alt="3- curl-headers">

* **4- wafw00f:**
screenshots/Recon/4-robots-txt.png
  <img src="screenshots/Recon/4-wafw00f.png" alt="4- wafw00f">

* **5- robots.txt and Nikto scan:**
  <img src="screenshots/Recon/5-robots-nikto.png" alt="5- robots.txt and nikto scan">


### M1: Initial Access

screenshots/M1/2-sqli-bypass.png

screenshots/M1/3-portal-access.png

### M2: Data Extraction


### M3: Critical Data Exposure

