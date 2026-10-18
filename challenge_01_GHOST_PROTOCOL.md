# Challenge 1: GHOST PROTOCOL

## Flag
```
OPSWAT{gh0st_pr0t0c0l_r3v34ls_th3_p4th_thr0ugh_dyn4m1c_r3v3rs1ng_8c71d4e92f6a13b
```

## Writeup

GHOST PROTOCOL 
 
Flag: 
OPSWAT{gh0st_pr0t0c0l_r3v34ls_th3_p4th_thr0ugh_dyn4m1c_r3v3rs1ng_8c71d4e92f6a13b
5} 
Approach (Step by Step):  
| Overview 
We get a small x86-64 ELF called ghost. It’s a text protocol with a ghost> prompt. You have 
to send the right sequence of commands, and at the end it prints a token. The strings in the 
binary show the commands hello, challenge, response, unlock and quit. There’s also 
an integrity check failed message, which looks suspicious. 
Step 1: Figure out the shape of the program 
The main loop reads a line with fgets and strips the newline with strcspn. It then 
dispatches on a state variable, not on the command alone. The state values I saw were 
0x13, 0x2b, 0x47 and 0x91. In each state, only one command is accepted. Each state 
compares the input against a fixed string: 
State 
Accepts 
Next state 
0x13 
hello 
0x2b 
0x2b 
challenge 
0x47 
0x47 
response <hex> 
0x91 
0x91 
unlock 
prints the token 
There is also a command counter that goes up on each accepted command. This matters 
later, because the counter feeds into the math. 
Step 2: The decoy 
Early in the loop, a line starting with # takes a separate path. That path compares 
0xdeadbeef (or 0xdeadbeef ^ 0x12345678 = 0xcc99e897 for the # case) against 
0x13579bdf. Neither value ever equals it, so the “integrity check failed” branch can’t be 
reached. It’s a decoy to waste time. I ignored it. 


========== PAGE 3 ==========

 
 
 
Step 3: The challenge and response 
On challenge, the program prints an 8-digit hex value. It comes from a small mixing 
function: 
python 
def mix(v): 
    v = rol32(v, 7) 
    v = (v - 0x61c88647) & MASK 
    v ^= v >> 13 
    v = v * 0x85ebca6b & MASK 
    v ^= v >> 16 
    return v 
The challenge is mix((counter + 0x28) ^ 0x2cedb7e5). The counter is 1 at that point, 
so the value is c488b60f. That matches what the binary printed. 
The response check uses the same mixer. The counter is now 2, so the value is: 
response = mix(challenge ^ 2 ^ 0xadd497a3) ^ 0xb94ae9af = 59ec4c2c 
The value is the same on every run, so no live interaction is needed. We can precompute the 
whole input. 
Step 4: The embedded flag 
unlock allocates two buffers with calloc and builds an 82-byte blob from three chunks in 
.rodata. The chunks are interleaved one byte at a time (a, b, c, a, b, c, …). They sit at file 
offsets 0x2180 (28 bytes), 0x2160 (27 bytes) and 0x2140 (27 bytes). The last byte left over 
from the first chunk goes at the end. 
The blob is then decoded with a rolling scheme. Each byte is XORed with a running key and 
rotated right by (i % 5) + 1. It’s XORed again with a salt, and both the key and the salt 
update after each byte: 
python 
key, salt = 0xa7, 0x31 
for i, b in enumerate(encoded): 
    v = b ^ key 
    key = (b + key + 0x3d) & 0xff 
    r = i % 5 + 1 
    v = ((v >> r) | (v << (8 - r))) & 0xff 
    out.append(v ^ salt) 
    salt = (salt + 0x17) & 0xff 


========== PAGE 4 ==========

 
 
 
The result is a clean ASCII string that starts with OPSWAT{ and ends with }. That is a good sign 
that the offsets and constants are right. 
Step 5: Solution 
Send these four lines: 
hello 
challenge 
response 59ec4c2c 
unlock 
$ printf 'hello\nchallenge\nresponse 59ec4c2c\nunlock\n' | ./ghost 
... 
[+] token: 
OPSWAT{gh0st_pr0t0c0l_r3v34ls_th3_p4th_thr0ugh_dyn4m1c_r3v3rs1ng_8c71d4e
92f6a13b5} 
solve.py reproduces the challenge value, the response and the flag straight from the 
binary’s bytes. 
Takeaways 
• 
The program is a state machine, so I mapped the states and transitions before reading 
any of the math. 
• 
The counter is part of the math. Using the wrong counter value (1 vs 2) gives the wrong 
response. 
• 
# and integrity check failed are red herrings. 
• 
The flag isn’t stored in plain text. It’s split across three chunks, interleaved and 
encoded, so strings won’t find it. 
 
 
 
 
 


========== PAGE 5 ==========
