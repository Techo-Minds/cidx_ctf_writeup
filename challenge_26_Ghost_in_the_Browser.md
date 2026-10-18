# Challenge 26: Ghost in the Browser

## Flag
```
OPSWAT{th3_br0ws3r_r3m3mb3r5_m0r3_th4n_y0u_th1nk_9c71e4a82f6d30b5}
```

## Writeup

Ghost in the Browser 
Flag: OPSWAT{th3_br0ws3r_r3m3mb3r5_m0r3_th4n_y0u_th1nk_9c71e4a82f6d30b5} 
Approach (Step by Step): 
1. Challenge Overview & Bundle Analysis: The challenge provides a package containing 
'browser_state.json', 'network.har', and 'app_bundle.js'. Analyzing app_bundle.js reveals an 
application storing encrypted vault data in indexedDB, while WebAuthn credentials serve as a decoy. 
2. Browser State & Secrets Extraction: From browser_state.json, we extracted stored localStorage 
parameters: 'credential_id': 'Q1RGLWNyZWRlbnRpYWwtMjAyNi0xMGFiY2RlZjAxMjM0NTY3ODk', 
'user_handle': 'sameer-vault-user-7f31', 'rp_id': 'vault.local', alongside indexedDB ciphertext, salt 
('bWlzYy1tZWRpdW0tc2FsdC0yMDI2LTAx'), and IV ('d2ViYXV0aC1naG9zdC1pdg=='). 
3. Cryptographic Key Derivation & Decryption: The JavaScript logic computes raw = credential_id 
+ ':' + user_handle + ':' + rp_id, hashes it with SHA-256, and feeds it into HKDF-SHA256 with the 
extracted salt and info string 'ghost-browser-vault' to derive a 256-bit AES-GCM key. Replicating this 
derivation in Python and executing AES-256-GCM decryption on the indexedDB ciphertext revealed the 
internal payload: {"vault":"internal","message":"Browser state is part of the authentication 
boundary.","flag":"OPSWAT{th3_br0ws3r_r3m3mb3r5_m0r3_th4n_y0u_th1nk_9c71e4a82f6d30b5}"}. 
 
 
 
 
 
 
 
 
 
 
 
 
 
 


========== PAGE 73 ==========
