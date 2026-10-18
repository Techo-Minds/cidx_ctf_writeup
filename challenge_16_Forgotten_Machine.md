# Challenge 16: Forgotten Machine

## Flag
```
OPSWAT{1_gu3ss_y0u_kn0w_4b0u7_br41nfuck}
```

## Writeup

Forgotten Machine 
Flag:  
OPSWAT{1_gu3ss_y0u_kn0w_4b0u7_br41nfuck} 
 
 
Approach (Step by Step): 
Event: OPSWAT CIDX CTF 2026 
Category: Cryptography 
Author: NotAnHacker 
Challenge File: challenge.txt 
Flag: OPSWAT{1_gu3ss_y0u_kn0w_4b0u7_br41nfuck} 
Challenge Description 
Something strange was recovered from an old machine. It doesn't look like a normal 
message, but the machine seems to understand it perfectly. Can you figure out what 
language it speaks and recover what it's trying to say? 
Solution 
We were given a text file named challenge.txt containing a long sequence of unusual 
characters. 
At first, it looked like random symbols, but after inspecting the file, I noticed that it contained 
only these characters: 


========== PAGE 53 ==========

 
 
 
+ - < > [ ] . 
These characters are instructions used by Brainfuck, an esoteric programming language. 
This matched the challenge description, which mentioned an old machine understanding a 
strange language. 
Step 1 — Identifying Brainfuck 
Brainfuck is a minimal programming language that uses eight instructions to manipulate 
memory and produce output. 
The program in challenge.txt contained 687 bytes of Brainfuck instructions. 
Instead of trying to decode the characters manually, I wrote a simple Python interpreter to 
execute the program. 
code = open("challenge.txt").read() 
 
code = [c for c in code if c in "+-<>[].,"] 
 
jump = {} 
stack = [] 
 
for i, c in enumerate(code): 
    if c == "[": 
        stack.append(i) 
    elif c == "]": 
        j = stack.pop() 
        jump[i] = j 
        jump[j] = i 
 
tape = [0] * 30000 
ptr = 0 
i = 0 
output = "" 
 
while i < len(code): 
    c = code[i] 
 
    if c == "+": 
        tape[ptr] = (tape[ptr] + 1) % 256 
    elif c == "-": 
        tape[ptr] = (tape[ptr] - 1) % 256 
    elif c == ">": 


========== PAGE 54 ==========

 
 
 
        ptr += 1 
    elif c == "<": 
        ptr -= 1 
    elif c == ".": 
        output += chr(tape[ptr]) 
    elif c == "[" and tape[ptr] == 0: 
        i = jump[i] 
    elif c == "]" and tape[ptr] != 0: 
        i = jump[i] 
 
    i += 1 
 
print(output) 
Step 2 — Analyzing the Output 
Running the interpreter produced another encoded string: 
=t%^Bc]qa{f*pmXdJUw*g0_jc>D)ns[A3%\bN/6)9Wp\Y@+ZMz 
The output was not readable English and did not contain the expected OPSWAT{} flag format. 
However, it consisted of printable ASCII characters, including letters, numbers, and symbols. 
This suggested that the Brainfuck program was only the first layer. 
After checking the character pattern, I identified the second encoding as Base92. 
Step 3 — Decoding Base92 
Base92 is a binary-to-text encoding scheme that uses printable ASCII characters to represent 
binary data. 
I used the following Python script to decode the Brainfuck output: 
s = r'=t%^Bc]qa{f*pmXdJUw*g0_jc>D)ns[A3%\bN/6)9Wp\Y@+ZMz' 
 
def val(c): 
    if c == "!": 
        return 0 
    if "#" <= c <= "_": 
        return ord(c) - ord("#") + 1 
    if "a" <= c <= "}": 
        return ord(c) - ord("a") + 62 
    raise ValueError(c) 


========== PAGE 55 ==========

 
 
 
 
values = [val(c) for c in s] 
 
bits = "" 
i = 0 
 
while i + 1 < len(values): 
    n = values[i] * 91 + values[i + 1] 
    bits += format(n, "013b") 
    i += 2 
 
if i < len(values): 
    bits += format(values[i], "06b") 
 
decoded = bytes( 
    int(bits[k:k+8], 2) 
    for k in range(0, len(bits) - 7, 8) 
) 
 
print(decoded.decode()) 
After decoding the Base92 string, I recovered the flag. 
Final Flag 
OPSWAT{1_gu3ss_y0u_kn0w_4b0u7_br41nfuck} 
Conclusion 
This challenge involved two layers of encoding. 
The first step was recognizing that the strange symbols in challenge.txt were actually 
Brainfuck instructions. Executing the program produced another encoded string. 
The second step was identifying that output as Base92 and decoding it to recover the flag. 
The challenge was straightforward once the programming language was identified, but the 
additional encoding layer made it more interesting. 
Tools Used: Python, Brainfuck interpreter, Base92 decoder. 
Solving Path: 


========== PAGE 56 ==========

 
 
 
challenge.txt → Brainfuck → Base92 → Flag
