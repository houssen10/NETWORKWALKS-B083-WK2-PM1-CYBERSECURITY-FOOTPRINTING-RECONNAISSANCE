# B083 Cybersecurity Footprinting and Reconnaissance

**Evidence date:** 2026-09-15
**Assessment type:** Passive public-domain checks plus an authorized private lab scan
**Primary public target:** `networkwalks.com`
**Private lab scope:** `10.0.0.0/24`

## Purpose and authorization

This lab demonstrates a small, reproducible reconnaissance workflow: collect publicly available DNS, registration, HTTP, technology, and WAF indicators for `networkwalks.com`, then inspect a deliberately scoped private lab network. The public-domain checks were passive or minimal requests (DNS lookups, WHOIS, one HTTP header request, and fingerprinting tools). The `10.0.0.0/24` Nmap/Zenmap scan was performed against the private lab network shown in the evidence.

This report is educational documentation, not permission to scan or probe unrelated systems. Perform reconnaissance only with explicit authorization, keep requests minimal, respect rate limits and terms of use, and stop if an owner asks. No credentials are included in this report. The screenshots are unmodified evidence captures; they may contain ordinary public response metadata such as a cookie value or contact address, so do not treat them as reusable secrets.

## Scope

| Scope | Included activity | Boundary |
| --- | --- | --- |
| `networkwalks.com` | DNS resolution and enumeration, WHOIS lookup, HTTP headers, WhatWeb, and WAF identification | Public information and minimal requests only |
| `10.0.0.0/24` | Zenmap quick scan (`nmap -T4 -F 10.0.0.0/24`) | Private lab network only |
| Other systems | None | Do not extend these commands to third-party networks without written authorization |

## Methodology and command examples

The captures were collected on Kali Linux. The exact tool versions were not recorded in every screenshot; the output itself is the source of truth for this lab.

```bash
nslookup networkwalks.com
dnsrecon -d networkwalks.com
whois networkwalks.com
curl -I https://networkwalks.com
whatweb networkwalks.com
wafw00f networkwalks.com
```

For the private lab only, Zenmap displayed the equivalent quick-scan command:

```bash
nmap -T4 -F 10.0.0.0/24
```

The process was intentionally staged from low-impact discovery to service enumeration:

1. Resolve the domain and collect DNS records.
2. Review registration and status information.
3. Request headers only, then fingerprint the public web stack.
4. Identify the reported WAF.
5. Scan the authorized private `/24` and record only responsive hosts and ports.
6. Interpret the observations defensively without claiming exploitation or compromise.

## Findings

### DNS and registration

`nslookup` returned `192.232.216.135` for `networkwalks.com` using the resolver shown in the capture (`8.8.8.8`).

The `dnsrecon` capture reported eight records, including:

| Record | Observation from evidence |
| --- | --- |
| SOA / NS | HostGator nameservers `ns6135.hostgator.com` and `ns6136.hostgator.com` |
| A | `networkwalks.com` -> `192.232.216.135` |
| MX | `mail.networkwalks.com` with priority `10` |
| TXT | A Google site-verification value and an SPF record beginning `v=spf1` |
| SRV | Multiple `_autodiscover._tcp.networkwalks.com` records pointing to cPanel autodiscovery hosts over port `443` |

The WHOIS capture identified GoDaddy.com, LLC as registrar, showed creation in 2019, a registrar registration expiration in 2027, and the statuses `clientTransferProhibited`, `clientUpdateProhibited`, `clientRenewProhibited`, and `clientDeleteProhibited`. It also reported unsigned DNSSEC. WHOIS data is time-sensitive and should be rechecked before operational decisions.

### HTTP and application fingerprinting

The header-only request returned `HTTP/2 200`. The visible headers included:

- `Server: Apache` and `Content-Type: text/html; charset=UTF-8`;
- WordPress-related indicators, including `X-Nginx-Cache: WordPress`;
- a `Set-Cookie` for the WPDM client with `Secure` and `HttpOnly` attributes;
- `Link` headers exposing the WordPress JSON API (`/wp-json/`) and an API page endpoint;
- `Referrer-Policy: no-referrer-when-downgrade`;
- `X-Content-Type-Options: nosniff`;
- a permissions policy and cache-related headers.

The header capture is evidence of observable response metadata, not proof that any endpoint is vulnerable.

WhatWeb reported Apache, WordPress `7.1`, Bootstrap `7.1`, jQuery `3.7.1`, WordPress Download Manager `3.3.58`, Google Tag Manager, an email address, JSON/JSON-LD indicators, and the title `NetworkWalks Academy`. The two WhatWeb screenshots show the same style of fingerprint output in separate captures.

`wafw00f networkwalks.com` identified the site as being behind a **ModSecurity (SpiderLabs) WAF** after two requests. This is a fingerprinting result, not a bypass attempt.

### Private lab network

Zenmap reported two hosts up from the `10.0.0.0/24` quick scan:

| Host | Evidence-backed result |
| --- | --- |
| `10.0.0.1` | Open TCP `631/ipp`, `3306/mysql`, `5000/upnp`, `5432/postgresql`, and `8080/http-proxy`; 95 other TCP ports were shown as closed |
| `10.0.0.2` | Host was up, but the scanned ports were in ignored states; 100 filtered TCP ports showed no response |

The scan completed in 14.93 seconds according to Zenmap. The topology view shows the two lab hosts and does not establish any relationship beyond the captured scan visualization.

## Defensive interpretation

The public observations suggest a WordPress site hosted behind Apache and a ModSecurity WAF, with WordPress Download Manager and several browser-facing integrations. Defenders should keep WordPress, plugins, JavaScript libraries, Apache, and WAF rules maintained; minimize version and platform disclosure where practical; review JSON API exposure and cookie scope; monitor DNS/MX/TXT/SRV changes; and protect administrative and download-management paths with strong authentication and least privilege.

For the lab network, database and proxy-like services on `10.0.0.1` are useful inventory findings but should not be assumed vulnerable. Confirm that services are required, bind them to trusted interfaces, restrict access with host and network firewalls, use encryption and authentication, and monitor unexpected listeners. A filtered result for `10.0.0.2` means only that the scan received no response from the tested ports; it does not prove the host is secure or has no services.

## Limitations

- Evidence is a point-in-time snapshot captured on 2026-09-15. DNS, WHOIS, HTTP headers, software versions, and WAF behavior can change.
- The public checks did not attempt authentication, exploitation, content discovery, vulnerability validation, or WAF bypass.
- A quick Nmap scan tests only the selected common ports and cannot establish that all services are absent.
- Product and version strings are fingerprints and may be incomplete or inaccurate; validate them through authorized configuration or asset-management sources.
- No conclusion about compromise, exploitability, or ownership can be drawn from these observations alone.

## Evidence index

Every screenshot below is copied unchanged into the repository root. Links are relative to this README.

| Evidence | Demonstrates |
| --- | --- |
| [nslookup-resolution.png](nslookup-resolution.png) | `nslookup networkwalks.com` resolving to `192.232.216.135` |
| [dnsrecon-records.png](dnsrecon-records.png) | HostGator NS, SOA, A, MX, TXT, and SRV observations |
| [whois-record.png](whois-record.png) | Registrar, dates, domain status, nameservers, and unsigned DNSSEC |
| [curl-headers.png](curl-headers.png) | HTTP/2 response headers, Apache, WordPress, WPDM cookie, JSON API links, and security-related headers |
| [whatweb-fingerprint-1.png](whatweb-fingerprint-1.png) | WhatWeb technology fingerprint and public metadata |
| [whatweb-fingerprint-2.png](whatweb-fingerprint-2.png) | Second WhatWeb capture corroborating the fingerprint output |
| [wafwoof-modsecurity.png](wafwoof-modsecurity.png) | WAFW00F identification of ModSecurity (SpiderLabs) |
| [zenmap-quick-scan.png](zenmap-quick-scan.png) | Zenmap quick scan results for `10.0.0.1` and `10.0.0.2` |
| [zenmap-topology.png](zenmap-topology.png) | Zenmap topology visualization for the private lab hosts |

## Conclusion

The workflow collected a restrained public footprint for `networkwalks.com` and a bounded service inventory for the authorized `10.0.0.0/24` lab. The evidence is sufficient to document DNS and registration metadata, the visible Apache/WordPress/WPDM stack, ModSecurity detection, and the lab host/port observations. It is not a vulnerability assessment and must not be used to justify scanning systems outside the stated authorization boundary.
