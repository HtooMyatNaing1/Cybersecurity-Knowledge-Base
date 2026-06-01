# Passive Recon Cheat Sheet

## WHOIS

whois example.com

## RDAP

curl -s https://rdap.verisign.com/com/v1/domain/example.com | jq .

## A Record

dig example.com A

## AAAA Record

dig example.com AAAA

## MX Record

dig example.com MX

## TXT Record

dig example.com TXT

## SOA Record

dig example.com SOA

## All Records

dig example.com ANY

## Subdomains via CT Logs

https://crt.sh/?q=%.example.com

## DNSDumpster

https://dnsdumpster.com

## Shodan

hostname:example.com

org:"Company Name"

port:443

http.component:"wordpress"

## Google Dorks

site:example.com

site:\*.example.com

filetype:pdf site:example.com

intitle:"index of"

## GitHub

org:company

filename:.env

password

apikey

token

## LinkedIn

site:linkedin.com/company

site:linkedin.com/in

## Useful OSINT Tools

- Shodan
- Censys
- crt.sh
- DNSDumpster
- SecurityTrails
- WHOXY
- GitHub
- Google
- LinkedIn

## Passive = No Direct Interaction

✓ WHOIS

✓ RDAP

✓ DNS Lookups

✓ CT Logs

✓ DNSDumpster

✓ Shodan

✓ GitHub

✓ LinkedIn

✓ Google

## Active = Direct Interaction

✗ Ping

✗ Nmap

✗ Port Scan

✗ Directory Bruteforce

✗ Social Engineering

✗ Web Fuzzing
