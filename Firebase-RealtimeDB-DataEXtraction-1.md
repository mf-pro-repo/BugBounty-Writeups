# Cloud Database Misconfiguration: Spatial Telemetry Leak
Vulnerability: Unauthenticated Firebase Realtime Database Read

Target: Confidential (AR/XR Infrastructure)

Status: Closed / Sanitized

Classification: CWE-284 (Improper Access Control)

Triage Result: Duplicate

## Synopsis
During a targeted reconnaissance of an organization’s Augmented Reality (AR) and Extended Reality (XR) infrastructure, a security misconfiguration was identified in the backend cloud database instances.

Several production subdomains were found to be utilizing Firebase Realtime Databases with permissive security rules (".read": true).  
This allowed any unauthenticated actor to extract the entire database tree, including internal project identifiers, infrastructure metadata, and sensitive spatial telemetry, via a simple REST API call.

## Researcher's Notes:

While the organization correctly restricted write access (preventing data tampering), the read exposure on an AR/XR platform is significant.

In a standard web app, a leaked database might just be user IDs. In an AR/XR environment, the database contains the digital twin of the physical world. This includes coordinate systems, rotation data (Quaternions), and project-specific "anchors."
For an adversary, this is the first step in mapping out how a platform "sees" and "interacts" with its environment.

## Tactical Impact
The following data categories were successfully extracted without any authentication:

  **Spatial Telemetry:** Exposure of `relativePosition` and `relativeQuaternion` data, revealing the technical logic used for positioning objects in 3D space.

  **Infrastructure Metadata:** Disclosure of internal project naming conventions (e.g., [REDACTED]-ar-[PROGRAM]) and environment structures.

  **Service Topology:** Identification of specific Firebase instances mapped to public-facing subdomains, allowing for further targeted reconnaissance.

## Discovery
[![FoxHunt](https://img.shields.io/badge/Tool-FoxHunt_v5.0-orange)](https://github.com/mf-pro-repo/Foxhunt)  
During Foxhunt's secretfinding phase, it discovered a firebase API key along with several firebase project endpoints. This was my attack vector.  
It's not uncommon for companies to push these "cool" or "immersive" tools quickly because HR is all about putting stuff out as quickly as possible. Because of this, it's not uncommon to find misconfigurations like this.  

That's why I always poke and prod at firebase services when I can. I always try read/write at a minimum.  

In this case we had the endpoints from our endpoint discovery and had the API key from our secretfinder, so we had everything we needed.  

## Proof of Concept (PoC)
The vulnerability was identified by inspecting the client-side configuration objects initialized by the web application.

  **1. Configuration Discovery:**
The application’s source code exposed a global configuration block containing the ```databaseURL```
```Javascript
// Sanatized example of the exposed client-side config
window._CONFIG_ = {
  PLATFORM_CONFIG: {
    databaseURL: "https://[REDACTED]-[PROJECT].firebaseio.com/",
    apiKey: "[REDACTED]"
  }
};
```
  **2. Unauthorized Data Extraction (Read):**
By appending `.json` to the root of the `databaseURL`, the Firebase REST API returned the full data structure without requiring a token or session.
```bash
curl -s "https://[REDACTED]-ar-[PROJECT].firebaseio.com/.json"
```

  **3. Integrity Verification (Write):**
To confirm the scope, a PUT request was attempted to verify if write access was also misconfigured.
```bash
curl -X PUT -d '{"researcher": "[MYHANDLE]"}' "https://[REDACTED]-ar-[PROJECT].firebaseio.com/test_node.json"
# Result: 403 Forbidden (Permission denied)
```
This confirmed the issue was strictly an Information Disclosure (Read) rather than a Data Destruction/Integrity (Write) risk.  

## Findings Summary
  **Affected Subdomains:** 3 unique AR/XR production environments.

  **Exposed Data:** Full JSON structure of the spatial backend.

  **Root Cause:** Failure to implement server-side authorization rules for read requests.

## Evidence

(I'll add screenshots eventually...)

## Remediation Recommendations
  **Update Security Rules:** Modify Firebase Realtime Database rules to deny public read access. At a minimum, implement `auth != null` to require a valid Firebase Authentication token.

  **Asset Sanitization:** Review client-side code to ensure internal environment naming conventions are not exposed in public-facing configuration objects.

## Takeaways

This finding was identified as a duplicate, but with confirmation that it was identified as a valid security issue so I count this one as a win.

A misconception among some is that a Firebase apiKey acts as a secret. In reality, Firebase API keys are identifiers, not authorizers.  
Their purpose is to tell the Firebase SDK which project to talk to, not to prove the user has permission to see the data.
This finding proves that relying on "Security through Obscurity" (hiding the URL in a JS bundle) is not a substitute for robust Firebase Security Rules.

Not every API key is "OMG I FOUND A SECRET REPORT IT" In fact, MOST are public keys or organizational identifiers. Check your work and verify verify verify...

This finding also highlights the importance of manual review and reinforces why FoxHunt is built on a "Hybrid" model. 
While automation (Grep/Pattern Matching) was used to find the haystack (the API keys), it took manual analysis to identify the needle (the unauthenticated .json REST exploit).

Bottom Line: If your cloud database doesn't require a token to read the root node, you aren't just hosting data, you're broadcasting it.

## Disclaimer
This write-up has been strictly sanitized to remove all proprietary identifiers, including hostnames and project-specific namespaces.  
All research was conducted following responsible disclosure guidelines and reported through official channels.  
Don't do bad things.
