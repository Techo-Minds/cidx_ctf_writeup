# Challenge 18: Broken Crown

## Flag
```
OPSWAT{jwt_4lg0r1thm_c0nfus10n_b3h1nd_k3y_r3us3_91c7e4b26d}
```

## Writeup

Broken Crown 
Flag: OPSWAT{jwt_4lg0r1thm_c0nfus10n_b3h1nd_k3y_r3us3_91c7e4b26d}  
 
 
Approach (Step by Step):  
Event: OPSWAT CIDX CTF 2026 
Category: Cryptography / Web 
Author: ADXD3459 
Challenge Files: token.txt, public.pem, check.py 
Flag: OPSWAT{jwt_4lg0r1thm_c0nfus10n_b3h1nd_k3y_r3us3_91c7e4b26d} 
Challenge Overview 
We were given three files: a signed JWT token, an RSA public key, and a Python script that 
verifies tokens. 
The original JWT used the RS256 algorithm and contained the role viewer. Our goal was to 
create a valid token with admin privileges and use it to recover the flag. 
After inspecting the verification script, I found that it supported both RS256 and HS256. The 
problem was that it used the same RSA public key for both algorithms. 
This made the checker vulnerable to JWT algorithm confusion. 
Step 1 — Inspecting the JWT 
I started by examining the provided token and its structure. 


========== PAGE 61 ==========

 
 
 
A JWT consists of three Base64URL-encoded parts: 
header.payload.signature 
The original token used RS256, meaning its signature was generated using an RSA private key 
and verified using the corresponding public key. 
The token also contained a viewer role, which was insufficient to access the protected 
content. 
Since we only had the public key, generating a new valid RS256 signature was not possible. 
Step 2 — Analyzing the Verification Script 
Next, I inspected check.py to understand how the token was validated. 
The script accepted two algorithms: 
• 
RS256 — verified using the RSA public key. 
• 
HS256 — verified using HMAC-SHA256. 
The vulnerability was in the HS256 verification process. 
Instead of using a separate secret key for HMAC, the script used the contents of 
public.pem. 
This was a major issue because RSA public keys are publicly available. 
The checker also validated several JWT fields: 
kid  : bc-signing-2026 
iss  : broken-crown.local 
aud  : bc-client 
role : admin 
To pass verification, the forged token needed these values, a valid subject, and an unexpired 
timestamp. 
Step 3 — Understanding the Vulnerability 
The attack was possible because the verifier trusted the algorithm specified in the JWT 
header. 
Normally: 
• 
RS256 uses an RSA private key for signing and a public key for verification. 


========== PAGE 62 ==========

 
 
 
• 
HS256 uses the same secret key for both signing and verification. 
However, the checker allowed us to change the algorithm to HS256 and use the publicly 
available RSA key as the HMAC secret. 
This meant we could generate our own valid signature without knowing the RSA private key. 
Step 4 — Forging an Admin Token 
I created a new JWT header using HS256 while keeping the expected key ID. 
{ 
  "typ": "JWT", 
  "alg": "HS256", 
  "kid": "bc-signing-2026" 
} 
I then created a payload with admin privileges. 
{ 
  "sub": "admin", 
  "role": "admin", 
  "scope": "*", 
  "iss": "broken-crown.local", 
  "aud": "bc-client", 
  "iat": 1781740800, 
  "exp": 4102444800 
} 
The expiry was set far enough into the future to satisfy the checker's expiration validation. 
Step 5 — Generating the JWT Signature 
I used Python to construct the token and sign it using HMAC-SHA256. 
import base64 
import hashlib 
import hmac 
import json 
 
def b64url(data): 
    return base64.urlsafe_b64encode(data).rstrip(b"=") 
 
header = { 
    "typ": "JWT", 


========== PAGE 63 ==========

 
 
 
    "alg": "HS256", 
    "kid": "bc-signing-2026" 
} 
 
payload = { 
    "sub": "admin", 
    "role": "admin", 
    "scope": "*", 
    "iss": "broken-crown.local", 
    "aud": "bc-client", 
    "iat": 1781740800, 
    "exp": 4102444800 
} 
 
header_encoded = b64url( 
    json.dumps(header, separators=(",", ":")).encode() 
) 
 
payload_encoded = b64url( 
    json.dumps(payload, separators=(",", ":")).encode() 
) 
 
signing_input = header_encoded + b"." + payload_encoded 
 
with open("public.pem", "rb") as f: 
    key = f.read() 
 
signature = hmac.new( 
    key, 
    signing_input, 
    hashlib.sha256 
).digest() 
 
token = signing_input + b"." + b64url(signature) 
 
with open("forged-token.txt", "w") as f: 
    f.write(token.decode()) 
 
print(token.decode()) 
The script generated a valid HS256 JWT using the RSA public key as the HMAC secret. 


========== PAGE 64 ==========

 
 
 
Step 6 — Testing the Forged Token 
After generating the token, I submitted it to the provided offline checker. 
python3 check.py forged-token.txt 
The checker accepted the forged token because its signature matched the HMAC calculated 
using public.pem. 
Since the token also contained the required admin role and metadata, the checker revealed 
the protected payload. 
The payload contained the flag. 
Final Flag 
OPSWAT{jwt_4lg0r1thm_c0nfus10n_b3h1nd_k3y_r3us3_91c7e4b26d} 
Conclusion 
This challenge demonstrated a JWT algorithm-confusion vulnerability caused by incorrect 
key handling. 
The verifier supported both RSA and HMAC signatures but reused the RSA public key as the 
HMAC secret. 
By changing the JWT algorithm from RS256 to HS256, I was able to forge an admin token and 
recover the flag. 
The important lesson is that JWT verification must enforce the expected signing algorithm 
rather than trusting the algorithm supplied by the token itself. 
Tools Used: Python, Base64URL encoding, HMAC-SHA256, JWT analysis. 
Solving Path: 
Original JWT → Inspect Checker → Identify Algorithm Confusion → Forge 
HS256 Admin Token → Verify → Flag
