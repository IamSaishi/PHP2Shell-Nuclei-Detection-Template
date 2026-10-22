# PHP2Shell Nuclei Detection Template

A custom [Nuclei](https://github.com/projectdiscovery/nuclei) template for identifying PHP versions that may be affected by the PHP2Shell vulnerabilities **CVE-2026-109618** and **CVE-2026-109619**.

The template performs lightweight HTTP fingerprinting by inspecting the `X-Powered-By` response header and matching disclosed PHP versions against configured version ranges.

> [!WARNING]
> This template performs version-based detection only. It does not exploit either vulnerability and cannot confirm remote code execution (RCE). Findings should be treated as potential exposure until the target's runtime and patch status have been verified.

## Overview

| Property | Details |
|---|---|
| Tool | Nuclei |
| Language | YAML |
| Detection type | HTTP version fingerprinting |
| Vulnerabilities | CVE-2026-109618, CVE-2026-109619 |
| HTTP method | GET |
| Paths checked | `/index.php`, `/` |
| Primary fingerprint | `X-Powered-By` |
| Exploitation | None |
| Authentication | Not required for the configured checks |

## How It Works

The template follows four steps:

1. **HTTP requests** — sends GET requests to `/index.php` and `/` on the target.
2. **Header inspection** — looks for a PHP version disclosed through `X-Powered-By`.
3. **Version matching** — compares the reported version against the configured ranges.
4. **Finding generation** — reports a match when a disclosed version falls within one of those ranges.

For example, a response containing:

```http
HTTP/1.1 200 OK
X-Powered-By: PHP/8.4.26
```

matches the configured PHP 8.4 range.

This indicates a potentially affected version was disclosed. It does not establish that the application is exploitable.

## Detection Coverage

The current template checks the following version ranges:

| PHP branch | Configured range | Detection category |
|---|---|---|
| PHP 8.5 | 8.5.0–8.5.11 | Potentially affected |
| PHP 8.4 | 8.4.0–8.4.26 | Potentially affected |
| PHP 8.3 | 8.3.0–8.3.35 | Potentially affected |
| PHP 8.2 | 8.2.0–8.2.34 | Potentially affected |
| PHP 8.1 | All reported 8.1.x versions | Legacy branch |
| PHP 8.0 | All reported 8.0.x versions | Legacy branch |
| PHP 7.x | All reported 7.x versions | Legacy branch |
| PHP 5.x | All reported 5.x versions | Legacy branch |

**Important:** These are configured detection ranges, not independent confirmation that every listed release is vulnerable to both CVEs. Verify affected releases, fixed versions, and vendor-specific backports against the official advisories before using the results for vulnerability reporting.

PHP 4.x and earlier are not covered by the current legacy matcher.

## Requirements

- Nuclei v3.x
- Network access to the target HTTP or HTTPS service
- Permission to assess the target

## Installation

### Option 1: Clone this repository

Replace `YOUR_USERNAME` with your GitHub username after publishing the repository.

```bash
git clone https://github.com/YOUR_USERNAME/php2shell-nuclei.git
cd php2shell-nuclei
```

### Option 2: Download the template

Download `php2shell-version-detect.yaml` from the repository and save it locally.

## Usage

### Validate the template

Validate the YAML before scanning:

```bash
nuclei -validate -t php2shell-version-detect.yaml
```

### Scan a single target

```bash
nuclei \
  -t php2shell-version-detect.yaml \
  -u https://target.example
```

### Scan a local test environment

```bash
nuclei \
  -t php2shell-version-detect.yaml \
  -u http://127.0.0.1:8088
```

### Scan multiple authorized targets

Create a file named `targets.txt`:

```text
https://target-one.example
https://target-two.example
https://target-three.example
```

Then run:

```bash
nuclei \
  -t php2shell-version-detect.yaml \
  -l targets.txt
```

Only scan systems you own or are explicitly authorized to assess.

## Example Detection Scenario

Given the following HTTP response:

```http
HTTP/1.1 200 OK
X-Powered-By: PHP/8.4.26
```

The PHP 8.4 matcher should match the configured range.

A typical Nuclei finding will identify the template and target URL. Exact output depends on the installed Nuclei version and output settings.

If no finding is returned, that does **not** prove the target is patched. The version may be hidden, the checked paths may not expose PHP, or the relevant application may use a different runtime configuration.

## Limitations

### 1. Header-dependent detection

The template relies on `X-Powered-By`. Servers that suppress or remove this header will not be detected by the current matchers.

### 2. Version fingerprinting is not exploit verification

A reported version may fall within a potentially affected range, but the actual runtime, build configuration, and reachability of the vulnerable code path still matter.

### 3. Vendor backports

Linux distributions and other vendors may backport security fixes without changing the upstream PHP version string. This can cause false positives.

### 4. Legacy PHP branches

The legacy matcher flags PHP 5.x, 7.x, 8.0, and 8.1 broadly. It should not be interpreted as proof that every version in those branches is affected by both vulnerabilities.

### 5. Limited endpoint coverage

Only `/index.php` and `/` are checked. Applications that expose PHP through other routes may not disclose a version at these paths.

## Recommended Validation Workflow

For an initial assessment:

1. Run the template against authorized targets.
2. Record the disclosed PHP version and response evidence.
3. Verify the installed PHP package version and security patch status where access is available.
4. Consult the official vulnerability advisories for affected releases and fixes.
5. Report results as potential exposure unless sufficient evidence establishes the vulnerability.

## References

- [PHP official release archive](https://www.php.net/archive/)
- [PHP source security advisory: GHSA-w8hw-w9fx-j79w](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w)
- [PHP source security advisory: GHSA-6wmf-4rv5-p3rp](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp)
- [ProjectDiscovery Nuclei](https://github.com/projectdiscovery/nuclei)
- [Nuclei template documentation](https://docs.projectdiscovery.io/templates/introduction)

## Disclaimer

This project is intended for authorized security testing, defensive research, and vulnerability triage.

The author is not responsible for misuse of this template. Always obtain appropriate authorization before scanning systems you do not own.
