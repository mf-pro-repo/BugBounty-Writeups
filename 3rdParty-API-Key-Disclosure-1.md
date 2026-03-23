# Third-Party API Key Exposure: Geolocation Services
Vulnerability: Hardcoded IPInfoDB API Key
Target: Confidential (Global Gaming/Entertainment Platform)

Status: Closed / Sanitized

Classification: CWE-798 (Use of Hardcoded Credentials)

Triage Result: Duplicate (Original report from 2024 marked "Informative")

## Synopsis
During a broad reconnaissance of a production web asset, a live IPInfoDB API key was identified within a publicly served JavaScript bundle. The key was found to be fully functional, allowing unauthenticated third parties to perform geolocation lookups on the organization's behalf.

While the organization previously assessed this as "Informative," the exposure represents a systemic failure in CI/CD secret management, as production credentials should never be bundled into client-side assets where they are subject to quota exhaustion and unauthorized usage.

## Researcher's Notes: The Cost of "Informative"
Organizations often triage third-party keys as low risk because they don't grant access to internal user data. However, this ignores the Operational Availability risk.

If this key is used for critical business logic—such as regional content gating, currency localization, or compliance checks—an attacker can simply script 10,000 requests to exhaust the API quota. This effectively "blinds" the application to its users' locations, breaking regional functionality across the entire platform. In security, Availability is just as important as Confidentiality.

On the flipside, if it's a public, free, or open source system, a leaked API key cannot be used for any malicious purpose but does still go against security best practices. (This is relevant and explained in the "Takeaways" section.)

## Discovery & Methodology
The Ideology: "Static Analysis at Scale"
My methodology assumes that the most vulnerable code isn't always the newest. Legacy scripts, often labeled main.js or common.js, frequently survive multiple development cycles without a security audit. These "forgotten" files are where hardcoded credentials go to hide.

1. Infrastructure-Wide Secret Hunting (FoxHunt)
Using the FoxHunt pipeline, I performed a recursive search across all collected JavaScript assets for the organization. I used a custom regex library designed to identify common API key patterns (Google, Firebase, IPInfo, AWS, etc.).

```bash
# Automated secret identification in the FoxHunt pipeline
grep -rE "api_key|apiKey|key=" ./collected_assets/*.js
```
2. Manual Asset Auditing
Automation flagged a match in a legacy script: /js/comeetMain.js. Upon manual inspection, the key was not just present but embedded directly into an API call string.

3. Live Validation
To confirm the key was not a "dummy" or "development" value, I performed a live request against the third-party provider's API. The return of a 200 OK with valid geolocation data for a test IP (8.8.8.8) confirmed the credential was active and billable.

## Proof of Concept (PoC)
1. Credential Identification:
The key was found embedded in a hardcoded URL string within the public JS bundle.
```bash
curl -s "https://[TARGET]/js/[REDACTEDFILE].js" | grep -o "ipinfodb[^'\"]*"
# Result: ipinfodb.com/v3/ip-city/?key=[REDACTED_KEY]&format=json
```
2. Unauthorized API Usage (The "Smoke Test"):
Confirmed the key works unauthenticated from any external IP address.
```bash
curl -s "https://api.ipinfodb.com/v3/ip-city/?key=[REDACTED_KEY]&ip=8.8.8.8&format=json"
```
3. Response Verification:
```json
{
    "statusCode": "OK",
    "ipAddress": "8.8.8.8",
    "countryName": "United States of America",
    "cityName": "Mountain View"
}
```

## Evidence

(One day I'll get around to screenshots)


## Tactical Impact
  **Quota Exhaustion:** Attackers can intentionally burn through the organization's paid API tier, causing a Denial of Service (DoS) for geolocation-dependent features.

  **Financial Risk:** If the API is "Pay-as-you-go," unauthorized usage directly impacts the organization's monthly billing.

  **CI/CD Failure:** This finding confirms that secrets are being committed to the codebase and bundled into production builds without automated secret scanning (e.g., Gitleaks/Trufflehog) in the pipeline.

## Takeaways
This report was closed as a duplicate of a 2024 finding. The fact that the key remained live for over two years highlights a remediation gap.

When you find a hardcoded key:

Don't just report the string. Show that it works.

Highlight the operational risk. Explain how breaking that specific API affects the end-user experience (e.g., "The site won't know which currency to display").

Check the age. If a bug has been there for two years, it's a sign of a larger "Secret Management" process failure.

For this one I was kind of dumb, I was unfamiliar with ipinfodb and their free API service. So resource exhaustion and pay-as-you-go were out of there. In reality the business impact was next to 0. I jumped the gun on this report but hey, it was like 4AM and I'll chock it up to lessons learned. It all goes back to verify verify verify. 

## Disclaimer
This write-up has been strictly sanitized to remove all proprietary identifiers. All research was conducted following responsible disclosure guidelines and reported through official channels. Don't do bad things.
