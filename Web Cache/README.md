### Web Cache Poisoning DoS

**How it works:** Send a single request with a malicious HTTP header (like `X-Forwarded-Port: 123` or `Transfer-Encoding: invalid`) that causes an error response. The cache stores this error and serves it to all subsequent visitors, creating a persistent DoS with just one request.

### Cache Poisoning + XSS

**Technique:** Researcher added cacheable file extensions (`.js`, `.css`) to URLs that normally wouldn't cache. Found that both a Cookie value and the `X-Forwarded-For` header were reflected in the response's JavaScript code, allowing XSS injection that bypassed WAF detection.

**WAF bypass trick:** Used 3 separate `X-Forwarded-For` headers to craft the payload across multiple reflections points, closing the `<script>` tag via Cookie while injecting the actual XSS via headers. The obfuscated payload avoided WAF triggers on common XSS patterns.

**Example request:**
```
GET /xxx/xx/xxx.xx/x.js?cache=key HTTP/2 
Host: Redacted
X-Forwarded-For: xss 
X-Forwarded-For: xss><svg/onload=globalThis[`al`+/ert/.source]`1`// X-Forwarded-For: > 
Cookie: gdId=xss</script%20
```
