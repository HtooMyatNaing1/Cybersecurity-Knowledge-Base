# Shodan

## Purpose

Shodan is a search engine for internet-connected devices and services.

Unlike Google, which indexes websites, Shodan indexes:

- Web servers
- Routers
- Firewalls
- Databases
- IoT devices
- Industrial control systems
- Cameras
- VPN gateways

## Common Use Cases

### Attack Surface Discovery

Identify public-facing systems belonging to a target organization.

### Technology Identification

Discover:

- Web servers
- Operating systems
- Services
- Software versions

### Exposure Assessment

Find:

- Exposed databases
- Open RDP servers
- Open SSH servers
- Misconfigured services

### Vulnerability Research

Identify software versions that may contain known CVEs.

---

# Useful Filters

## Country

country:US

country:TH

## Port

port:22

port:80

port:443

port:3389

## Organization

org:"Amazon"

org:"Google"

## Hostname

hostname:example.com

---

# Useful Searches

## Apache Servers

apache

## Specific Apache Version

apache 2.4.49

## Exposed SSH

port:22

## Exposed RDP

port:3389

## Jenkins

http.title:"Dashboard [Jenkins]"

## Elasticsearch

product:"Elasticsearch"

---

# Pentesting Workflow

Target
→ Shodan
→ Discover Hosts
→ Identify Services
→ Research CVEs
→ Validate Exposure
→ Document Findings

---

# Notes

- Shodan data may be old.
- Always verify findings manually.
- Never assume a vulnerable version is actually exploitable.
- Use Shodan as a reconnaissance tool, not proof of vulnerability.
