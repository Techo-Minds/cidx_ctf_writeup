# Challenge 6: Hidden in Plain Sight

## Flag
```
OPSWAT{h1dd3n_1n_7h3_l0g0_n0_b0dy_kn0ws}
```

## Writeup

Hidden in Plain Sight 
Flag:  
OPSWAT{h1dd3n_1n_7h3_l0g0_n0_b0dy_kn0ws} 
 
Approach (Step by Step):  
Overview 
The challenge suggests that a familiar symbol contains information that is not visible at first 
glance. It also identifies the organizers as an important starting point and specifically hints at 
steganography. 
The CTF is organized as the OPSWAT CIDX CTF, so the first step was to locate the official 
CIDX logo used on the challenge website. 
Locating the Image 
The organizer logo was found at: 
https://opswatcidx.ctfd.io/files/a9b3356d447d10e007bd8fef3ca0d456/CIDX-Logo.jpg 
At first inspection, the image appeared to be an ordinary JPEG logo. There was no visible text 
or suspicious content in the picture itself. 
Since the challenge mentioned steganography, the file was inspected for hidden data rather 
than relying only on its appearance. 
Steganography Analysis 
The image was checked with normal file and metadata tools: 


========== PAGE 21 ==========

 
 
 
file CIDX-Logo.jpg 
exiftool CIDX-Logo.jpg 
Nothing immediately useful was found in the visible image or metadata. Since JPEG files are 
commonly used with steghide, an extraction attempt was made: 
steghide extract -sf CIDX-Logo.jpg 
The extraction required a passphrase. To recover it, stegseek was used with the RockYou 
wordlist: 
stegseek CIDX-Logo.jpg rockyou.txt 
The correct passphrase was found to be: 
cutiesmile 
After successful extraction, a file named flag.txt was recovered from the image. 
Extracted Flag 
The contents of flag.txt were: 
OPSWAT{h1dd3n_1n_7h3_l0g0_n0_b0dy_kn0ws} 
 
 
 
 
 
 
 
 
 
 
 


========== PAGE 22 ==========
