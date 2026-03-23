# Production Telemetry Injection & Infrastructure Leak

Vulnerability: Exposed Production Config (Firebase & Sentry DSN)
Target: Confidential (E-Commerce / Ad-Platform)

Status: Closed / Sanitized

Classification: CWE-212 (Exposure of Sensitive Information)

Triage Result: Not Applicable (Contested)

## Synopsis
During reconnaissance of a high-traffic Retail Ad-Platform, a publicly accessible configuration file (`config.js`) was identified. This file contained active production credentials for both Firebase and Sentry (Error Monitoring).

While the Firebase instance had correctly configured security rules for data access, the Sentry DSN (Data Source Name) was confirmed to be globally writable.
This allowed me, a pretend bad guy, to inject arbitrary error events into the production monitoring pipeline, potentially masking real incidents and exhausting resource quotas.

## Researcher's Notes: The "Alert Fatigue" Attack
Organizations often dismiss Sentry DSN leaks because they don't grant "read" access to data. This is a narrow view of security.

As an adversary, if I can write to your error logs, I can:

  **Trigger False Positives:** Flood your Slack/PagerDuty with thousands of fake "Critical" errors to mask a real attack occurring simultaneously.

  **Resource Exhaustion:** Burn through your monthly Sentry event quota in minutes, leaving the platform blind to genuine production crashes.

  **Internal Mapping:** Use the "Authorized Domains" list linked to the API key to find "shadow" infrastructure (Preprod/Beta) that isn't indexed by search engines.

## Tactical Impact
**1. Confirmed Sentry Injection (Write Access)**
The exposed DSN was not merely a "leak" but a functional write-token. I successfully injected a custom security-research event into the production environment.

**Impact:** Ability to pollute dashboards, trigger false alerts, and disrupt the Sentry telemetry pipeline.

**2. Infrastructure Mapping (Information Disclosure)**
By querying the Google Identity Toolkit with the leaked Firebase API Key, I extracted a list of Authorized Domains. This revealed internal infrastructure targets that were previously unknown:

`beta-[REDACTED].io (Legacy/Beta asset)`
`[REDACTED]-dashboard-consumer.preprod.[REDACTED].dev (Internal Preproduction environment)`

## Discovery

During FoxHunt's JS endpoint discovery phase, the tool flagged `https://[SUBDOMAIN].[TARGET].com/config.js`.

My standard methodology involves auditing any `config.*` files found during recon. While these are often sanitized, they are frequently overlooked during rapid deployment cycles. A simple curl request confirmed that the file was served as a static asset without any authentication requirement.

## Proof of Concept (PoC)
**1. Discovery & Credential Retrieval**
The credentials were found in a static JS file served without any authentication.
```bash
curl -s https://[TARGET_DOMAIN]/config.js
# Returns window.config with Firebase and Sentry credentials
```
**2. Sentry Event Injection (The "Smoking Gun")**
To prove exploitability beyond "Information Disclosure," I sent a crafted error event to the production Sentry ingest endpoint.
```bash
curl -s -X POST "https://[SENTRY_INGEST_URL]/api/[PROJECT_ID]/store/" \
  -H "Content-Type: application/json" \
  -H "X-Sentry-Auth: Sentry sentry_version=7, sentry_key=[EXPOSED_KEY]" \
  -d '{
    "event_id": "a1b2c3d4e5f6789012345678abcdef01",
    "level": "error",
    "message": "Security Research PoC - [HANDLE]",
    "platform": "javascript"
  }'
# Response: {"id":"a1b2c3d4e5f6789012345678abcdef01"} (Success)
```

## Findings Summary
Exposed Asset: config.js

**Key Status:** Confirmed Valid (Firebase & Sentry)

**Injected Event ID:** `a1b2c3d4e5f6789012345678abcdef01`

**Internal Targets Leaked:** 2 (Beta and Preprod environments)

## Evidence

(More screenshots to add)

## Takeaways
This report was closed as "Not Applicable" by the organization, under the premise that there was "no direct security implication."
I can see where they are coming from but in modern DevSecOps, Telemetry Integrity is just as important as Data Confidentiality.
If an attacker can manipulate your monitoring, they own your perspective of the system's health.  
I could hide actual malicious traffic in a flood of fabricated traffic, I could exhaust resources or piss off the SOC team to the point they start ignoring findings as "false positives"  
But as I said, I can see where they are coming from, I am a but more paranoid than others and I try to see exploitability in almost everything.

Pro-Tip for Researchers: When reporting "Exposed Keys," always show a side-effect. I showed I could write to their logs—if you can't show a side-effect, the triagers will almost always hit the "N/A" button.

## Disclaimer
This write-up has been strictly sanitized. All research was conducted following responsible disclosure guidelines and reported through official channels.
Don't do bad things.
