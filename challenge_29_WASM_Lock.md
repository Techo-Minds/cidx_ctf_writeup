# Challenge 29: WASM Lock

## Flag
```
OPSWAT{st4t1c_4n4lys1s_r3v34ls_th3_s3cr3t_but_th3_fl4g_1s_l0ng_7f92a1c84e6b39d5}
```

## Writeup

WASM Lock 


========== PAGE 83 ==========

 
 
 
 
Flag: 
OPSWAT{st4t1c_4n4lys1s_r3v34ls_th3_s3cr3t_but_th3_fl4g_1s_l0ng_7f92a1c84e6b39d5} 
Approach (Step by Step): 
WASM Lock 
 
**Category:** Reverse Engineering   
**Difficulty:** Easy (as presented in the challenge) 
 
## Objective 
 
Analyze the provided `lock.wasm` WebAssembly module to understand its hidden 
verification mechanism and recover the flag through static analysis. 
 
## Methodology 
 
### Step 1: Extract the challenge file 
 
Extract `lock.wasm` from the supplied archive and identify it as a 
WebAssembly module. 
 
### Step 2: Perform static analysis 
 
Inspect the WebAssembly module's sections, exported functions, code, and 
embedded data. Exported verification functions can reveal how input is 
checked, while data sections may contain obfuscated strings or other clues. 
 
Tools commonly used for this task include `wasm-objdump`, `wasm2wat`, and a 
WebAssembly-aware decompiler. 
 
### Step 3: Investigate the verification logic 
 
Examine the verification function and any transformations applied to the 
input or embedded data. If the code indicates a fixed XOR transformation, 
for example, test that transformation against the relevant bytes. 
 
Illustrative Python snippet (the XOR key below is an example, not a verified 
property of this challenge): 
 
```python 
decoded = bytes(b ^ 0x5A for b in data) 
print(decoded) 


========== PAGE 84 ==========

 
 
 
``` 
 
 
### Step 4: Recover and validate the flag 
 
Use the instructions and embedded data recovered from the binary to 
reconstruct the expected flag. Validate the result against the challenge's 
required flag format.
