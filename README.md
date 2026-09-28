# Credential Phishing Kit Investigation

## Executive Summary

This project documents an end-to-end investigation of a credential-phishing campaign targeting users within a controlled financial-services lab environment.
The investigation began with suspicious payment-themed emails and progressed beyond traditional email triage into analysis of an HTML attachment, Base64 deobfuscation, redirect infrastructure, a Microsoft credential-harvesting page, exposed adversary infrastructure, and the underlying phishing kit.

Analysis of the phishing kit identified captured credential records and server-side PHP logic configured to transmit compromised information to an external email address.

The investigation demonstrates the correlation of email, web, file, threat-intelligence, and source-code evidence to understand the complete phishing attack chain and develop actionable indicators for detection and response.

---

## Investigation Scope

The investigation focused on:

- Email and attachment analysis
- MIME and Base64 analysis
- HTML redirect analysis
- Credential-harvesting infrastructure
- Phishing kit acquisition and examination
- SHA-256 file hashing
- Threat intelligence using VirusTotal
- Captured credential analysis
- PHP source-code analysis
- Indicator of Compromise (IOC) extraction
- MITRE ATT&CK mapping
- SOC containment and response recommendations

---

## Attack Chain

```text
Phishing Email
      ↓
HTML Attachment
      ↓
Base64-Encoded Payload
      ↓
HTML META Redirect
      ↓
External Phishing Infrastructure
      ↓
Microsoft Credential-Harvesting Page
      ↓
Credential Submission
      ↓
Server-Side PHP Processing
      ↓
Credential Collection / Exfiltration
```

---

## Email & Attachment Analysis

The campaign used a payment-themed email originating from:

`Accounts.Payable@groupmarketingonline[.]icu`

One message contained an HTML attachment named:

`Direct Credit Advice.html`

Rather than relying on the attachment's displayed content, the raw email source was examined.

MIME analysis identified:

`Content-Transfer-Encoding: base64`

The encoded HTML payload was extracted and decoded using CyberChef.

Analysis of the decoded HTML identified an automatic META refresh redirecting the browser to infrastructure hosted under:

`kennaroads[.]buzz`

This established the connection between the email attachment and the external phishing infrastructure.

---

## Credential-Harvesting Infrastructure

The redirect destination presented a Microsoft-branded authentication page requesting the user's password.

Although the page visually impersonated Microsoft, it was hosted on the unrelated domain:

`kennaroads[.]buzz`

This confirmed that the infrastructure was being used for credential harvesting.

---

## Phishing Kit Discovery

Further investigation of the exposed web infrastructure identified an accessible archive:

`Update365.zip`

The archive was downloaded into the controlled analysis environment and fingerprinted using SHA-256.

### SHA-256

```text
ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
```

The hash was subsequently investigated using VirusTotal.

Archive metadata identified **49 contained files**, including PHP, JavaScript, CSS, images, and other supporting resources associated with the phishing infrastructure.

---

## Credential Capture Analysis

An exposed server-side log contained credential submissions and associated metadata.

The records included information such as:

- Email address
- Client IP
- User agent
- Country
- Timestamp

Analysis showed repeated credential submission associated with one affected account, providing evidence that the phishing infrastructure was actively recording victim input.

Sensitive credential values have intentionally been excluded from this public repository.

---

## Phishing Kit Source-Code Analysis

The phishing kit was extracted and its PHP components were reviewed.

Static analysis of `submit.php` identified a PHP `mail()` function configured to transmit captured information to an external email address.

The identified collection address was:

`m30pat@yandex[.]com`

This provided an additional adversary-controlled indicator directly from the phishing kit's server-side logic.

---

## Indicators of Compromise

| Type | Indicator |
|---|---|
| Sender | `Accounts.Payable@groupmarketingonline[.]icu` |
| Phishing Domain | `kennaroads[.]buzz` |
| Attachment | `Direct Credit Advice.html` |
| Phishing Kit | `Update365.zip` |
| SHA-256 | `ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686` |
| Credential Collection Email | `m30pat@yandex[.]com` |

---

## MITRE ATT&CK Mapping

| Technique | ID | Tactic |
|---|---|---|
| Spearphishing Attachment | T1566.001 | Initial Access |
| Input Capture: Web Portal Capture | T1056.003 | Credential Access |

---

## Recommended SOC Response

Following confirmation of credential compromise, recommended response actions include:

- Reset credentials for confirmed affected users
- Revoke active authentication sessions and tokens
- Review identity-provider sign-in logs for anomalous access
- Block identified phishing domains, URLs, and sender infrastructure
- Search enterprise mailboxes for related campaign messages
- Identify additional users who visited the phishing infrastructure
- Review affected accounts for malicious mailbox rules and forwarding
- Preserve relevant email, web, identity, and endpoint evidence
- Escalate confirmed account compromise through established incident-response procedures

---

## Tools & Technologies

- Email message-source analysis
- CyberChef
- Linux CLI
- SHA-256
- VirusTotal
- Web infrastructure analysis
- PHP source-code analysis
- MITRE ATT&CK

---

## Conclusion

The investigation established a complete credential-phishing chain from email delivery through credential collection.

Rather than relying on a single indicator, the assessment correlated email artifacts, decoded HTML, redirect infrastructure, the credential-harvesting page, phishing-kit files, threat intelligence, captured records, and server-side source code.

The investigation demonstrates how analysis of exposed adversary infrastructure can extend a phishing investigation beyond the original email and produce additional intelligence for detection, containment, and threat hunting.

---

> **Environment Note:** This investigation was conducted in a controlled, non-production cybersecurity lab. Sensitive credential values are intentionally excluded from the public repository.
