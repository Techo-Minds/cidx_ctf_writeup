# Challenge 7: Chimera

## Flag
```
OPSWAT{ch1m3r4_runt1m3_tr4c1ng_r3v34ls_th3_s3cr3t_but_th3_r34l_ch4ll3ng3_1s_f1n
```

## Writeup

Chimera 
Flag: 
OPSWAT{ch1m3r4_runt1m3_tr4c1ng_r3v34ls_th3_s3cr3t_but_th3_r34l_ch4ll3ng3_1s_f1n
d1ng_th3_f4k3s_7b91d42e6c80a5f3}  
 
 
Approach (Step by Step): Overview 
chimera.exe implements a legacy authentication system. The executable is unusually large 
because it contains a 600 MiB resource that is checked for integrity before authentication is 
attempted. 
The actual authentication routine is small. It transforms every input byte and compares the 
result against a fixed byte table stored inside the executable. 
The challenge can be solved through static analysis without running the supplied program. 
1. Inspecting the Executable 
The extracted file is a 64-bit Windows PE executable with a size of approximately 629 MB. 
Most of the file is occupied by the resource section: 
.text  — approximately 31 KB 
.rsrc  — approximately 600 MiB 
Several useful strings are present in the executable: 
PROJECT CHIMERA 
Legacy authentication subsystem v4.7 
Runtime integrity verification enabled. 
Enter access key: 
[+] Authentication successful. 
[+] Vault access granted. 


========== PAGE 23 ==========

 
 
 
Cross-references to these strings identify the main program routine at: 
0x140008750 
The main routine performs the following actions: 
1. Checks the embedded resource. 
2. Reads the authentication input. 
3. Passes the input to the key-checking function. 
The relevant functions are: 
Main routine:          0x140008750 
Resource checker:      0x140001550 
Authentication checker: 0x140001480 
2. Resource Integrity Check 
The program looks for resource ID 101 with resource type 10, which corresponds to RCDATA. 
The resource must have the following size: 
0x25800000 bytes 
This is exactly: 
629,145,600 bytes 
600 MiB 
The resource begins at file offset: 
0xA858 
The executable calculates a CRC32 value using the polynomial: 
0xEDB88320 
The internal accumulator begins with: 
0xFFFFFFFF 
The expected internal accumulator is: 
0xB14B4F6D 


========== PAGE 24 ==========

 
 
 
After applying the conventional final CRC32 complement, the expected CRC32 value 
becomes: 
0x4EB4B092 
Calculating the CRC32 of the resource produces: 
0x4EB4B092 
Therefore, the original resource is valid and does not need to be patched or removed. 
3. Reversing the Authentication Routine 
The authentication routine requires an input length of: 
0x6F bytes 
This is equal to: 
111 bytes 
The routine initializes the following values: 
Rolling checksum = 0x4C3A2B1D 
First mask       = 0x17 
Second mask      = 0 
For each byte at index i, the input byte is transformed using: 
encoded = (((input_byte ^ (0x17 + 17 * i)) * 13) ^ (7 * i)) & 0xFF 
The resulting value is compared with a fixed 111-byte table stored at virtual address: 
0x14000A1A0 
The corresponding file offset is: 
0x81A0 
4. Reversing the Transformation 
The transformation is reversible. 
• 
XOR can be undone by applying XOR again. 
• 
Multiplication by 13 is invertible modulo 256 because 13 is odd. 
• 
The modular inverse of 13 modulo 256 is 197. 


========== PAGE 25 ==========

 
 
 
This is because: 
13 × 197 = 2561 
2561 mod 256 = 1 
Therefore, the original input byte can be recovered with: 
input_byte = (    (((table_byte ^ ((7 * i) & 0xFF)) * 197) & 0xFF)    ^ 
((0x17 + 17 * i) & 0xFF)) 
5. Recovering the Flag 
The following script extracts the comparison table, reverses the transformation, verifies the 
checksum, and validates the large resource. 
Save it as solve_chimera.py beside chimera.exe. 
from pathlib import Pathimport zlibpath = Path("chimera.exe")# The 
comparison table is located at file offset 0x81A0.with path.open("rb") 
as f:    f.seek(0x81A0)    table = f.read(111)assert len(table) == 111# 
Reverse the byte transformation.key = bytes(    (        (((value ^ ((7 
* i) & 0xFF)) * 197) & 0xFF)        ^ ((0x17 + 17 * i) & 0xFF)    )    
for i, value in enumerate(table))# Reproduce the original byte 
comparisons and rolling checksum.checksum = 0x4C3A2B1Dfor i, character 
in enumerate(key):    encoded = (        ((character ^ (0x17 + 17 * i)) 
* 13) ^ (7 * i)    ) & 0xFF    assert encoded == table[i]    checksum = 
(        ((checksum << 5) | (checksum >> 27)) & 0xFFFFFFFF    ) ^ 
(encoded + i)assert checksum == 0x04F2B9B7 
The recovered input passes: 
• 
All 111 byte comparisons. 
• 
The final rolling checksum: 
0x04F2B9B7 
• 
The resource integrity check: 
0x4EB4B092 
The decoded input is the flag. 
Final Flag 
OPSWAT{ch1m3r4_runt1m3_tr4c1ng_r3v34ls_th3_s3cr3t_but_th3_r34l_ch4ll3ng3
_1s_f1nd1ng_th3_f4k3s_7b91d42e6c80a5f3} 
 


========== PAGE 26 ==========
