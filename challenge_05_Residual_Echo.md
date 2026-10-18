# Challenge 5: Residual Echo

## Flag
```
OPSWAT{S0m3_Th1ng5_4r3_B3tt3r_H1dd3n}
```

## Writeup

Residual Echo 
Flag: OPSWAT{S0m3_Th1ng5_4r3_B3tt3r_H1dd3n} 
 
 
 
 
Approach (Step by Step):  
Challenge Description 
Sometimes, the smallest details hold the biggest secrets. 
Take a closer look at where you are, from where you started, think outside the box, and follow 
the clues. The answer might be closer than you think. 
The hint says: 
Every page has a story, but not everything is visible at first glance. Sometimes, it helps to look 
at things from a different perspective. 
This suggests checking the website source code, hidden HTML comments, and older 
versions of the page. 
Step 1: Inspect the Current Website 
The challenge points to: 
https://opswatcidx.ctfd.io/ 
Opening the current page and checking the visible content did not reveal the flag. 
The page source was also inspected for: 
• 
HTML comments 
• 
Hidden text 
• 
Unusual links 
• 
JavaScript variables 
• 
Encoded strings 


========== PAGE 18 ==========

 
 
 
• 
Suspicious metadata 
No useful flag was present in the current version of the page. 
Step 2: Interpret the Hint 
The important phrases in the hint are: 
• 
“Every page has a story” — check the page history. 
• 
“Not everything is visible” — inspect hidden HTML content. 
• 
“Different perspective” — use an archived copy or source view. 
Because the live page may have been modified after the challenge was created, an archived 
version was required. 
Step 3: Search the Internet Archive 
The Internet Archive CDX endpoint was checked for historical captures of the CTF homepage. 
An archived capture was found at: 
https://web.archive.org/web/20261016191354id_/https://opswatcidx.ctfd.io/ 
The id_ portion is useful because it returns the archived page content directly without the 
usual archive toolbar. 
Step 4: Inspect the Archived HTML 
The archived page was downloaded and inspected as raw HTML. 
A hidden HTML comment was found in the source. The comment was not visible when 
viewing the page normally because HTML comments are ignored by the browser. 
The comment contained a Pastebin link: 
https://pastebin.com/J1YiTB26 
This was the hidden trail left by the challenge author. 
Step 5: Retrieve the Pastebin Content 
The Pastebin page was opened and its raw content was checked. 


========== PAGE 19 ==========

 
 
 
The flag appeared directly in the Pastebin data: 
OPSWAT{S0m3_Th1ng5_4r3_B3tt3r_H1dd3n} 
The value was verified from the raw Pastebin response and matched the challenge’s required 
flag format. 
OPSWAT{S0m3_Th1ng5_4r3_B3tt3r_H1dd3n} 
Useful Commands 
The archived page could be inspected with: 
curl -L 
"https://web.archive.org/web/20261016191354id_/https://opswatcidx.ctfd.io/" -o 
archived.html 
Search the HTML for comments and Pastebin references: 
grep -inE "comment|pastebin|OPSWAT|hidden" archived.html 
Retrieve the Pastebin content: 
curl -L "https://pastebin.com/raw/J1YiTB26" 
 
 
 
 
 
 
 
 
 
 
 


========== PAGE 20 ==========
