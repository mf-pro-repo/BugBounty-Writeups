# BugBounty-Writeups

Security Research & Bug Bounty Archive
by nullf0x
This repository is a collection of my independent security research, vulnerability findings, and custom reconnaissance tooling. I focus on infrastructure-level flaws, edge-case bypasses, and systemic misconfigurations in enterprise environments.

## The Engine: Foxhunt
[![FoxHunt](https://img.shields.io/badge/Tool-FoxHunt_v5.0-orange)](https://github.com/mf-pro-repo/Foxhunt)  

Most of the findings in this repo were identified using Foxhunt, my custom stateful reconnaissance pipeline. It’s designed to handle the "boring" parts of recon—persistence, scope validation, and data deduplication, so I can focus on the fun part of exploitation.

  **Logic-Driven:** Built-in alerts for internal IP leaks and high-value service identification.
  
  **Resilient:** Session-based with checkpoint resume and clean interrupt handling.
  
  **Clean Data:** Automatically bifurcates public and private infrastructure to reduce scan noise.


## Vulnerability Writeups
Each writeup is sanitized to protect the organizations involved while maintaining full technical transparency for the research community.

(TODO - Add Table with links)


## Methodology
I don’t believe in "Spray and Pray." My research follows a deliberate, multi-stage pipeline to ensure every finding is high-signal and actionable.

  1: **Passive Aggregation:** Deep-diving into DNS records, client-side JS bundles, and public metadata to identify the target's external surface area.
  
  2: **Logic Filtering:** Using automation (like Foxhunt) to flag anomalies, such as RFC 1918 internal IP disclosures, suspicious headers, or misconfigured edge proxies.
  
  3: **Exploit Pathing:** This is the creative phase. Once an anomaly is flagged, I map out potential exploit chains—determining if a leak leads to SSRF, if a header allows for SSO smuggling, or if a hardcoded key grants unauthorized data access.
  
  4: **Active Validation:** Confirming the bug with a manual, controlled Proof-of-Concept (PoC) to verify impact before writing a single word of the report.
  
  5: **Sanitization:** Cleaning all receipts, logs, and screenshots to ensure they meet the highest responsible disclosure standards and protect the target's identity.

## Notes

You may notice in a lot of my reports I utilize googles DNS `8.8.8.8` I do this because I run a tailscale system and it REALLY messes up my DNS queries over CLI for wheatever reason. I could fix it, but honestly it's easier just to pass googles DNS where it can take it, plus Foxhunt has configurable DNS...


## Disclaimer
*This repository is for educational and authorized security testing purposes only. I do not condone or support unauthorized access to any systems. All findings reported here have been through the proper disclosure channels or were identified on public, properly scoped targets.*
