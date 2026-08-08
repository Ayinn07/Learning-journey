# Lab: File Path Traversal, Simple Case

## Summary
A Path Traversal vulnerability was identified in the image loading functionality of the web application. This flaw allows an unauthenticated remote attacker to manipulate file paths and read arbitrary files directly from the underlying server file system.

---

## Vulnerability Details
* **Vulnerability Type:** Path Traversal / Directory Traversal (CWE-22)
* **Severity:** High / Critical
* **CVSS v3.1 Score:** 7.5 (High) — `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`
* **Vulnerable Endpoint:** `GET /image?filename=`

---

## Impact
An attacker can exploit this vulnerability to read sensitive system files (such as `/etc/passwd`), potentially exposing server configuration details, internal system usernames, or sensitive application credentials.

---

## Steps to Reproduce

1. Open the target application in your web browser with traffic routed through a proxy tool (e.g., Burp Suite).
2. Access any blog post or page where images are loaded from the server. Observe that images are retrieved via a request similar to:
   ```http
   GET /image?filename=1.jpg HTTP/1.1
   Host: target.com
