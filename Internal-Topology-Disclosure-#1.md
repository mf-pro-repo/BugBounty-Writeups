# Internal Topology Disclosure 1

## Information:
Vulnerability: Internal Infrastructure Mapping via Public DNS  
Status: Completed / Sanitized  
Classification: CWE-200 (Information Exposure)  
Triage Result: Duplicate / Informative  


## Synopsis
During reconnaissance, a systemic misconfiguration was identified where numerous public-facing subdomains resolved to RFC 1918 (Private) IP addresses. 
While the company dismissed this as "Low Risk" due to the non-routable nature of the addresses. It provides an external actor with a high-fidelity map of the internal network architecture, including the location of critical services like Vault, GHE, and CI/CD pipelines.


## Researcher's Notes:  
This is the finding that made me add a full DNS lookup of all found hosts to FoxHunt.  
Organizations often view internal IP disclosure as a non-issue because "they can't be reached externally." I disagree.

I pride myself on thinking like an adversary, and discovering internal IPs provides a pre-made map of the target's internal environment.  
While this information is technically "post-infiltration" utility, it allows an attacker to skip the Discovery Phase of a campaign, traditionally one of the LOUDEST and most detectable parts of a breach.

**The Tactical Advantage:**  
  **Target Prioritization:** An attacker already knows exactly where the "Crown Jewels" reside before they even gain a foothold.

  **Infrastructure Mapping:** Depending on the disclosure size, an attacker can map infrastructure by subnet, service, or even physical location.

  **SSRF Weaponization:** These leaked IPs serve as a verified "hit list" for any Server-Side Request Forgery (SSRF) vulnerabilities found on the perimeter.

  **Stealth:** By removing the need for internal network scanning (ARP/Port sweeps), the attacker remains silent, moving directly to targeted exploitation.

## Discovery

[![FoxHunt](https://img.shields.io/badge/Tool-FoxHunt_v5.0-orange)](https://github.com/mf-pro-repo/Foxhunt)  
During basic recon of the target, subfinder found a series of domains. Doing my proper due diligence I ran dig on all of them to identify them properly.  
This is when I discovered they were private IP addresses and were linked to various services throughout the company. This then became my attack vector.

The identification of these assets was performed using passive DNS aggregation followed by a filtered resolution check against public name servers.  

  **1. Subdomain Aggregation:** Passive sources (Censys, crt.sh, and Subfinder) were used to generate a candidate list.  
  
  **2. Filtered Resolution:** The following logic was applied to flag hostnames resolving to RFC 1918 space  

```bash
# Example query against a public resolver
dig @8.8.8.8 +short dev-vault.target.com
# Result: 10.50.1.101
```

  **3. Bulk Identification (Foxhunt Stage 4):** Using the Foxhunt pipeline, the following command isolates internal leaks:  
  ```bash
  dnsx -l subdomains.txt -resp-only -a | grep -E "^(10\.|172\.(1[6-9]|2[0-9]|3[0-1])\.|192\.168\.)" 
  ```  

## Finding:

Overall 26 internal IP addresses were discovered spanning 24 different services.
I was able to determine their prod subnet at 10.36.XXX.XXX and their training subnet at 172.16.XXX.XXX
I even found one of their internal services was also mapped to an external IP and was publically accessable from anywhere on the internet. (Auth gated)  

Mapped Infrastructure Includes:  
  **Security Tooling:** Internal Portswigger platform.

  **Development & CI/CD:** TeamCity, GitHub Service, NIX Service.

  **Core Business Logic:** Machine Learning Services, Simulation Services, Mapping/Geo services.

  **Ops & Documentation:** JIRA, various Metrics/Telemetry services.

## Screenshots

(Insert screenshots here when I upload them)  

## Suggested Actions:  

  **Split-Horizon DNS:** Host internal records on internal-only name servers.  
  
  **DNS Sanitization:** Audit public DNS zones to ensure no RFC 1918 addresses are leaked during infrastructure-as-code deployments.  

## Takeaways  

The core lesson here is that visibility can be a vulnerability. Externally discoverable internal IPs act as a force multiplier for an adversary.  
In a typical breach, the "Network Discovery" phase is where the attacker is most vulnerable to detection; it is the point where an IDS (Intrusion Detection System) identifies anomalous scanning or lateral movement attempts.

By providing a map of your infrastructure via public DNS:

  **The Discovery Phase is bypassed:** An attacker doesn't need to "find" the targets; they already have the coordinates.

  **IDS/IPS Evasion:** Because there is no need for noisy network sweeps, the attacker remains below the threshold of most behavioral detection alerts.

  **Surgical Precision:** An adversary can move directly from initial foothold to the "Crown Jewels," significantly reducing the "Mean Time to Compromise."

## Disclaimer  
This repository is for educational and authorized security testing purposes only. I do not condone or support unauthorized access to any systems. All findings reported here have been through the proper disclosure channels or were identified on public, properly scoped targets.  

Yes it was reported. There is no information in this report that identifies the company.
