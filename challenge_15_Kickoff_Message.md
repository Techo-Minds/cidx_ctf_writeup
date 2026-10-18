# Challenge 15: Kickoff Message

## Flag
```
Flag: OPSWAT{y0u_f0und_7h3_h1dd3n_fl4g}
```

## Writeup

Kickoff Message 
Flag:  
Flag: OPSWAT{y0u_f0und_7h3_h1dd3n_fl4g} 
 
 
Approach (Step by Step):  
Category: Steganography / Cryptography 
Author: NotAnHacker 
Challenge File: message.txt 
Flag: OPSWAT{y0u_f0und_7h3_h1dd3n_fl4g} 
Challenge Overview 
We were given a text file named message.txt containing what looked like a normal kickoff 
announcement for OPSWAT CIDX CTF 2026. 
At first, there was nothing suspicious about the message: 
OPSWAT | CIDX CTF 2026 is LIVE!   
However, the file contained more data than the visible message suggested. 
Step 1 — Inspecting the File 
I started by checking the file type and size. 


========== PAGE 49 ==========

 
 
 
file message.txt 
wc -c message.txt 
The output showed that it was a UTF-8 text file with a size of 828 bytes. 
This was unusual because the visible message was very short. 
I suspected that the file contained hidden Unicode characters, so I used Python to inspect 
them. 
from collections import Counter 
 
text = open("message.txt", encoding="utf-8").read() 
 
for char, count in Counter(text).items(): 
    print(f"U+{ord(char):04X}: {count}") 
This revealed two interesting characters: 
U+200C: 161 
U+200D: 103 
Both are invisible Unicode characters. 
• 
U+200C — Zero Width Non-Joiner 
• 
U+200D — Zero Width Joiner 
There were 264 invisible characters in total, hidden after the flag emoji. 
Step 2 — Extracting the Hidden Data 
Since there were only two types of invisible characters, I tried treating them as binary digits. 
I mapped U+200C to 0 and U+200D to 1. 
text = open("message.txt", encoding="utf-8").read() 
 
bits = "" 
 
for char in text: 
    if char == "\u200c": 
        bits += "0" 
    elif char == "\u200d": 
        bits += "1" 
 
data = bytes( 


========== PAGE 50 ==========

 
 
 
    int(bits[i:i+8], 2) 
    for i in range(0, len(bits), 8) 
) 
 
print(data.hex()) 
This produced 33 bytes of data: 
783862241e604a17072a6c52430c590c6e4437076e06063b57071d265104051422 
The output wasn't readable text, which suggested that the hidden message was encrypted. 
Step 3 — Identifying the Encryption 
Since the flag format was already known to be OPSWAT{}, I tried XORing the first few 
ciphertext bytes with the expected flag prefix. 
known = b"OPSWAT{" 
 
key_part = bytes( 
    a ^ b for a, b in zip(data, known) 
) 
 
print(key_part) 
Output: 
b'7h1s_41' 
This looked like part of a readable key rather than random data. 
The challenge also had a key shared through the event's Discord server: 
7h1s_41n7_34sy 
The key translates to "this_aint_easy" in leetspeak. 
This confirmed that the message was encrypted using repeating-key XOR. 
Step 4 — Decrypting the Flag 
With the key identified, I wrote a small Python script to decrypt the ciphertext. 
key = b"7h1s_41n7_34sy" 
 


========== PAGE 51 ==========

 
 
 
flag = bytes( 
    byte ^ key[i % len(key)] 
    for i, byte in enumerate(data) 
) 
 
print(flag.decode()) 
Running the script gave: 
OPSWAT{y0u_f0und_7h3_h1dd3n_fl4g} 
The hidden message was successfully recovered. 
Final Flag 
OPSWAT{y0u_f0und_7h3_h1dd3n_fl4g} 
Conclusion 
This challenge combined Unicode steganography with basic XOR encryption. 
The main trick was noticing that the text file contained more data than the visible message. 
The invisible Unicode characters represented binary data, which could be converted into 
ciphertext. 
After identifying the XOR key, decrypting the message was straightforward. 
Tools Used: Python, Linux file, wc, Unicode inspection, XOR decoding. 
The challenge was a good reminder that even a simple text file can contain hidden 
information that isn't visible in a normal text editor. 
 
 
 
 
 
 
 


========== PAGE 52 ==========
