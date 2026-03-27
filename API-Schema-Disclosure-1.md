# Enterprise API Schema Exposure (Internal Topology Leak)

Vulnerability: Unauthenticated Protobuf Schema Disclosure

Status: Closed / Sanitized

Classification: CWE-200 (Information Exposure)

Triage Result: Duplicate / Informative ***(Bullshit)***

## Prelude  
This one is going to be kind of hard and is going to be even more vague than usual.  
Due to the sensativity of the information discovered and the companies service, a LOT has had to be sanitized and left vague but I'll do my best.  

## Synopsis
During reconnaissance of a production Industrial IoT (IIoT) and Resource Management platform, a significant information disclosure was identified. A publicly accessible, unauthenticated JavaScript bundle was found to contain the complete compiled Protobuf API schema for the platform’s internal microservices.

This schema provided a granular roadmap of the internal application logic, including RPC methods and message structures for device telemetry, hardware control, authentication flows, and data classification handlers.  

## Researcher's Notes: The "Blueprints" Argument
This finding highlights the critical distinction between Transport-layer security (how a user reaches a page) and Asset-layer exposure (what the page gives the user).

While this was orignally classified as a "Low Risk" informative finding, serving an entire internal API schema to a client-side bundle is a fundamental architectural failure. In a high-stakes enterprise environment, this data is the technical equivalent of a "Blueprints" leak.  
It allows an adversary to understand the internal workings of the system with near-perfect clarity before ever attempting an exploit.

And in this case it's still a vulnerability I'm pressing to the company, so the current severity or triage results may change.

## Tactical Impact & "The Map"
The leaked schema (spanning over 2,300 lines of definitions) enumerated several critical capabilities:

  **Hardware Control:** API structures for device state management, telemetry frequency, and remote command execution.

  **Service Topology:** A complete list of internal service names and their inter-dependencies, revealing the backend application architecture.

  **Authentication Infrastructure:** A mapping of the internal identity and access management (IAM) services, providing a roadmap for targeted credential or token-based attacks.

  **Data Handling Logic:** Defined structures for sensitive data categorization, confirming the platform's role in handling high-integrity information.  

## Discovery 
[![FoxHunt](https://img.shields.io/badge/Tool-FoxHunt_v5.0-orange)](https://github.com/mf-pro-repo/Foxhunt)  
During the JavaScript parsing phase of FoxHunt, this was not initially flagged by automation. It was identified through manual verification. (Automation is a force multiplier, but manual analysis is where the real bugs live.)

My reconnaissance process involves pulling and parsing every JS file a target provides. These are gold mines for secrets, internal functionality, management schemes, and hidden endpoints.
In this situation, I simply identified a disproportionately large JS file, opened it, and identified the serialized Protobuf definitions.

## Proof of Concept (PoC)
The disclosure was identified by analyzing static assets served via the platform's public-facing infrastructure.

  **1. Automated Retrieval:**
No authentication or session cookies were required to fetch the production bundle via standard HTTP requests.

```bash
# Sanitized retrieval command
curl -sk -H 'User-Agent: [REDACTED]' 'https://[TARGET]/assets/index-[HASH].js' > bundle.js
```

  **2. Schema Extraction:**
By parsing the internal namespace strings, I was able to reconstruct the internal API surface.  
```bash
# Extracting internal service namespaces
cat bundle.js | tr '"' '\n' | grep '[INTERNAL_NAMESPACE].' | sort -u
```

  **3. Discovery Logic:**
The host was identified via infrastructure-wide scanning. An initial 421 Misdirected Request from the edge proxy confirmed an active backend, which led to the discovery of the unauthenticated asset path.

## Findings Summary
Namespace Count: 2,300+ unique Protobuf definitions extracted.

Infrastructure: Service Mesh / Envoy Proxy.

Impact: Direct disclosure of the "Attack Surface" for a production industrial platform.

## Evidence

(Add screenshots here when I get them uploaded.)

## Takeaways
In modern web architecture, what you send to the client stays with the client. For a major infrastructure provider, the API schema is the "Key to the Kingdom." It allows an attacker to build a local mirror of the system, identify weak points in the logic offline, and craft surgical exploits without ever alerting production monitoring tools.

When you provide the map, you eliminate the attacker's need for discovery.

The "Duplicate" Logic
In this case, the organization handles assets of extreme value. It is baffling to classify this as a duplicate when the report it allegedly duplicates merely notes that an endpoint is available. One concerns access; the other concerns intelligence.

I don't make the rules, I just find ways to break them. This case is still under review.  
**Don't hand the bad guys the map.**

## Disclaimer

This write-up has been strictly sanitized to remove all proprietary identifiers, including hostnames, organization names, and project-specific internal namespaces.
The intent of this documentation is to highlight architectural patterns and systemic vulnerabilities, not to facilitate targeted attacks.  
All information was properly disclosed to the company through official channels.  
Don't do bad things.
