# Challenge 19: Welcome Flag

## Flag
```
OPSWAT{W3lc0m3_T0_OPSWAT_CIDX_CTF_2026}
```

## Writeup

Welcome Flag 


========== PAGE 65 ==========

 
 
 
Flag: OPSWAT{W3lc0m3_T0_OPSWAT_CIDX_CTF_2026} 
Approach (Step by Step): 
1. Reconnaissance of Standard Web Endpoints: Inspecting the root web application directory and 
standard crawler directives at '/robots.txt' revealed administrative path entries alongside hidden 
content embedded within the response body. 
2. Discovery of Obfuscated Payload: A careful examination of the whitespace and non-directive 
comments within the '/robots.txt' stream uncovered an isolated Base64-encoded string: 
'T1BTV0FUe1czbGMwbTNfVDBfT1BTV0FUX0NJRFhfQ1RGXzIwMjZ9'. 
3. Decoding & Flag Recovery: Decoding the ASCII payload from Base64 directly yielded the plain 
ASCII flag: OPSWAT{W3lc0m3_T0_OPSWAT_CIDX_CTF_2026}.
