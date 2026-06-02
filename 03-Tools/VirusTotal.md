# VirusTotal

## Purpose

VirusTotal aggregates results from multiple antivirus engines and threat intelligence providers.

You can analyze:

- Files
- URLs
- Domains
- IP addresses
- Hashes

---

# Common Use Cases

## Malware Analysis

Check whether a suspicious file has been detected by security vendors.

## Threat Intelligence

Investigate:

- Malicious domains
- IP addresses
- URLs
- File hashes

## IOC Validation

Validate Indicators of Compromise (IOCs).

---

# Detection Ratio

Example:

35/72

Meaning:

- 35 vendors detected the file
- 72 vendors scanned it

---

# Important Rule

0/72 detections does NOT mean safe.

It only means:

"Not detected by current engines."

---

# Useful Artifacts

VirusTotal may provide:

- Related domains
- Related IPs
- Download URLs
- DNS records
- Community comments
- Historical information

---

# Pentesting Workflow

Suspicious File
→ VirusTotal
→ Detection Ratio
→ Extract IOCs
→ Threat Intelligence Research
→ Documentation

---

# Notes

- Never upload confidential client files.
- Uploading a file may share it with security vendors.
- Always combine VirusTotal results with manual analysis.
