# Challenge 23: Shortlink

## Flag
```
OPSWAT{dns_rebind_ssrf_toctou_ftw}
```

## Writeup

Shortlink 
Flag: OPSWAT{dns_rebind_ssrf_toctou_ftw} 
Approach (Step by Step): 
1. Web Architecture & Session Analysis: The target application is a URL shortening service built on 
Python Flask. Session tokens are signed client-side cookies formatted as 
'{"role":"user","username":"..."}'. The admin panel at '/admin' is restricted by role validation. 
2. Session Secret Cracking & Role Forgery: Using flask-unsign with the rockyou dictionary, the 
Flask SECRET_KEY was cracked as 'budakbageur'. Forging a signed cookie with 
'{"role":"admin","username":"admin"}' granted full administrative access. 
3. Internal API Reconnaissance: Source comments in the admin interface revealed an internal 
backend service running locally at '127.0.0.1:5001'. The '/admin/create' endpoint creates shortlinks, 
and '/go/<code>' fetches the target URL server-side and renders the response, establishing an SSRF 
sink. Direct attempts to link to '127.0.0.1' were blocked by a creation-time DNS resolution check 
requiring a public IP. 
4. TOCTOU DNS Rebinding Exploitation: Because IP validation occurs only at creation time while 
the actual HTTP request executes dynamically upon visiting '/go/<code>', the system is vulnerable to 
Time-of-Check to Time-of-Use (TOCTOU) DNS rebinding. Using 'rbndr.us' configured to alternate 
between a public IP and 127.0.0.1, the link creation succeeded during the public resolution cycle, 
while subsequent server-side fetching resolved to 127.0.0.1:5001. 
5. Internal Flag Exfiltration: Querying the rebound endpoint routed the request to the internal 
administrative API at 'http://127.0.0.1:5001/flag', exfiltrating the response: 
OPSWAT{dns_rebind_ssrf_toctou_ftw}. 
 
 
 
 
 
 
 
 


========== PAGE 69 ==========
