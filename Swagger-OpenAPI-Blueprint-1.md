# Internal API Blueprint Exposure (Swagger UI)
Vulnerability: Unauthenticated Swagger UI / OpenAPI Disclosure

Status: Closed

Classification: CWE-212 (Exposure of Sensitive Information)

Triage Result: Duplicate

## Synopsis
During reconnaissance of a QA-tier web asset, an unauthenticated Swagger UI instance was identified on a linked Azure-hosted test backend. The exposure provides a complete OpenAPI specification for the organization's internal employee portal API.

This disclosure maps over 130+ internal endpoints, revealing sensitive administrative functions, employee PII routes, organizational structure logic, and internal AI (ChatGPT) integration schemas.

## Discovery & Methodology

[![FoxHunt](https://img.shields.io/badge/Tool-FoxHunt_v5.0-orange)](https://github.com/mf-pro-repo/Foxhunt)  

In a tiered environment (Dev -> QA -> Prod), the QA environment can sometimes be a weak link. It usually contains "Production-lite" features but lacks the robust authentication middleware found in the live environment. My methodology here focused on extracting environmental variables and hardcoded URLs from these middle-tier assets to find "Shadow" infrastructure. I hate auth gates so I try to bypass them whenever I can.

1. Endpoint Extraction (FoxHunt)
The discovery began by parsing the JavaScript bundles of the QA environment. I was looking specifically for URL patterns that deviated from the primary domain.
```bash
# Extracting backend references from client-side JS
curl -s "https://[QA-DOMAIN]/main.js" | grep -oP 'https://[REDACTED]-test[^"'\'']*'
```

2. Path Fuzzing & Enumeration  
Once the test backend `*.azurewebsites.net` was identified, I performed a targeted scan for common developer artifacts. Swagger UI is a high-priority target because it is often enabled by default in development builds of .NET and Java APIs.

3. Specification Analysis

The discovery of `/swagger/v1/swagger.json` allowed for a complete reconstruction of the internal API topology without sending a single intrusive probe to the endpoints themselves.



## Proof of Concept (PoC)
1. Discovering the Backend:  
The backend URL was extracted from a publicly served bundle on the QA site.
```bash
# Reconstructing the discovery command
grep -oP 'https://app-[REDACTED]-test[^"]*' bundle.js
```

2. Accessing the UI:  
The UI was accessible via browser at the standard path without any JWT or Session requirement.
`https://[REDACTED]-test.azurewebsites.net/swagger/index.html`

3. Retrieving the Full Spec:  
```bash
curl -s "https://[REDACTED]-test.azurewebsites.net/swagger/v1/swagger.json"
```

## Tactical Impact

| Endpoint | Data/Risk Category | Potential Impact |
| :--- | :--- | :--- |
| `/api/GetUserProfile` | **Employee PII** | Names, IDs, and Contact Info |
| `/api/GetDirectReports/` | **Org Structure** | Managerial hierarchies and reporting lines |
| `/api/dashboard/currentBudget`| **Financial Data** | Internal department budgets and allocations |
| `/api/chat-gpt/query` | **Internal AI Leak** | Internal prompts, AI logic, and integration points |
| `/api/cache/reset` | **Operational DoS** | Unauthorized cache manipulation |
| `/api/departments/files` | **Sensitive Documents**| Internal file management and directory structures |

## Evidence

(More screenshots I have to add, shut up I'm posting the writeups first.)

## Remediation Recommendations  
Environment-Aware Config: Use environment variables to strictly disable Swagger/OpenAPI middleware in non-development builds.

Network Gating: Restrict access to test/QA backends via IP Whitelisting or VPN-only access.

Code Sanitization: Ensure that production and QA bundles do not contain hardcoded references to internal-only test environments.

## Takeaways
I do a lot of poking around when I find stuff like this, because information disclosure is one thing, but unauthenticateded read/write is another.

In this case everthing was locked down enough that an attacker would have to be pretty persistent or have a pivot, but it's always good to check. 

Lastly, know your systems. I've used OpenAPI in my own projects in the past and I know how the UI and calls work. It made it MUCH easier for me to test for exploits and build my report. It's impossible to know every single system used by every target, but knowing a little bit about the big ones can help in the long run.

While this ended up not being readily exploitable, it goes back once again to "Don't give the bad guys a map". Hardening those API routes might stop random scanners and bug hunters but an attacker with motive to target your company will document all of it. If and when they manage to get in, they have everything they need.

## Disclaimer
This write-up has been strictly sanitized. All research was conducted following responsible disclosure guidelines and reported through official channels. Don't do bad things.
