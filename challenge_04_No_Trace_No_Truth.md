# Challenge 4: No Trace, No Truth

## Flag
```
OPSWAT{TH3_R3C0RD_W4S_N3V3R_L0ST_JU5T_BUR13D_B3Y0ND_WH4T_W3_C0ULD_533}
```

## Writeup

No Trace, No Truth 
  
Flag: 
OPSWAT{TH3_R3C0RD_W4S_N3V3R_L0ST_JU5T_BUR13D_B3Y0ND_WH4T_W3_C0ULD_533} 
 
 
Approach (Step by Step): Idea 
The handle led to a Substack note. An archived version pointed to a Catbox image, then a 
GitHub repository, then a hidden YouTube handle. The flag was visible in the channel's full 
banner image, outside YouTube's displayed crop. 
Step 1 — Follow the Substack hint 
The initial handle alone did not produce a useful code-host profile. A challenge 
announcement explicitly directed participants to Substack: 
https://substack.com/@st4rk17 
The current note suggested that old versions of changed or deleted material could survive. 
Public note metadata showed: 
Field 
Value 
Note ID 
354304965 
Posted 
6 October 2026, 15:01:17 UTC 
Edited 
6 October 2026, 15:05:08 UTC 
The note's direct address was: 
https://substack.com/@st4rk17/note/c-354304965 
This made an archived version a sensible next lead. The useful archived-note evidence was 
supplied as a screenshot during the investigation. A new capture of the current profile only 
preserved the current clue; it did not recover the old text. 
Archive lookup: 


========== PAGE 13 ==========

 
 
 
https://archive.ph/https://substack.com/@st4rk17/note/c-354304965 
Step 2 — Interpret the archived note 
The archived note mentioned a cat knocking over a box of photographs and a copy labelled: 
qecgr8 
The combination of cat and box suggested the Catbox file host. Testing the label as an image 
filename found: 
https://files.catbox.moe/qecgr8.png 
The PNG was a QR code, not an ordinary photograph. 
Step 3 — Decode the QR code 
For local reproduction, download the image and decode it with zxing-cpp and Pillow: 
curl -L 'https://files.catbox.moe/qecgr8.png' -o clue.png 
python -m pip install zxing-cpp pillowimport zxingcpp 
from PIL import Image 
 
for result in zxingcpp.read_barcodes(Image.open("clue.png")): 
    print(result.text) 
The decoded message directed us to a person called stankops, their storage, and fragments 
of the trail. 
Step 4 — Recover a deleted GitHub record 
The GitHub account had a relevant repository: 
https://github.com/stankops/some-fragments 
Clone it and inspect its history: 
git clone https://github.com/stankops/some-fragments.git 
git -C some-fragments log --all --oneline 
git -C some-fragments log --all -p 
Relevant commits: 
Commit 
Change 
9c9b8b5 
Initial README 


========== PAGE 14 ==========

 
 
 
6e00864 
Added record.txt 
e19f5b6 
Deleted record.txt 
af45395 
Updated the README 
Recover the deleted file from the commit that added it: 
git -C some-fragments show 6e00864:record.txt > record.txt 
The visible text described an abandoned project and suggested a hidden message. It was 
signed Jarvis. More importantly, the underlying text contained 67 invisible Unicode 
characters. 
Step 5 — Decode the zero-width message 
The character set matched StegZero's Standard format: 
U+200B U+200C U+200D U+2060 U+2062 U+2063 U+2064 U+FEFF 
Each symbol represents three bits. The recovered header was: 
a55a015fef000e39aa19a2 
Header field 
Value 
Magic 
0xA55A 
Version 
1 
Nonce 
0x5FEF 
Payload length 
14 bytes 
Expected CRC32 
0x39AA19A2 
The payload bits were obfuscated with a xorshift32 generator. The following decoder 
reproduces the recovery using only Python's standard library: 
from pathlib import Path 
import zlib 
 
alphabet = "\u200b\u200c\u200d\u2060\u2062\u2063\u2064\ufeff" 
text = Path("record.txt").read_text(encoding="utf-8") 
bits = "".join( 
    f"{alphabet.index(character):03b}" 
    for character in text 
    if character in alphabet 
) 
 
header = bytes(int(bits[i:i + 8], 2) for i in range(0, 88, 8)) 
assert header[:3] == bytes.fromhex("a55a01") 


========== PAGE 15 ==========

 
 
 
nonce = int.from_bytes(header[3:5], "big") 
length = int.from_bytes(header[5:7], "big") 
expected_crc = int.from_bytes(header[7:11], "big") 
 
state = (nonce << 16) | 0xA55A 
remaining = 0 
accumulator = 0 
plain_bits = [] 
payload_bits = bits[88:88 + length * 8] 
assert len(payload_bits) == length * 8 
 
for bit in payload_bits: 
    if remaining == 0: 
        if state == 0: 
            state = 0x9E3779B9 
        state ^= (state << 13) & 0xFFFFFFFF 
        state ^= state >> 17 
        state ^= (state << 5) & 0xFFFFFFFF 
        state &= 0xFFFFFFFF 
        accumulator = state 
        remaining = 32 
    plain_bits.append(str(int(bit) ^ (accumulator & 1))) 
    accumulator >>= 1 
    remaining -= 1 
 
plain_bits = "".join(plain_bits) 
message = bytes( 
    int(plain_bits[i:i + 8], 2) 
    for i in range(0, len(plain_bits), 8) 
) 
assert zlib.crc32(message) == expected_crc 
print(message.decode("utf-8")) 
Output: 
@tonystank-q5i 
The matching CRC32 verified that this was a correctly decoded message rather than a 
guessed handle. 
Step 6 — Inspect the YouTube channel's full banner 
The handle resolved to: 
https://www.youtube.com/@tonystank-q5i 


========== PAGE 16 ==========

 
 
 
Channel ID: 
UCp6LvDbe0cceQLECtKK7iZQ 
The channel had no public video entries in the retrieved feed. Its Jarvis avatar matched the 
theme of the recovered record, and its banner showed a pixel-art cityscape. 
YouTube displayed a narrow crop of the banner. The underlying Google-hosted image URL 
contained resizing and cropping parameters after its first =. Replacing that suffix with =s0 
retrieved the full 2560 × 1440 image. 
Full image URL used during the solve: 
https://yt3.googleusercontent.com/cxZy_pMU-A8YyVp-
LFcrQxvPkgVhiTgGGMEM6LNm5e2Jgk6imRdluYVDoB4SlVX1tlDTbMbA=s0 
Manual steps: 
1. Open the channel banner image in a new tab, or copy its image source through the 
browser inspector. 
2. Keep the image identifier before the first =. 
3. Replace the entire resizing/cropping suffix with =s0. 
4. Load the full image and inspect its bottom edge. 
The bottom of the original image contained: 
OPSWAT{TH3_R3C0RD_W4S_N3V3R_L0ST_JU5T_BUR13D_B3Y0ND_WH4T_W3_C0ULD_533} 
Preserve the uppercase lettering. R3C0RD, L0ST, B3Y0ND, and C0ULD use the digit zero. The 
final word is 533. 
 
 
 
 
 
 
 
 


========== PAGE 17 ==========
