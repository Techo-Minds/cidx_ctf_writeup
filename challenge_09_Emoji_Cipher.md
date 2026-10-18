# Challenge 9: Emoji Cipher

## Flag
```
OPSWAT{3m0j15_4r3_jU5t_4n07h3r_3nc0d1ng_l4y3r_bu7_p4773rn5_4lw4y5_l34k_7f92a1
```

## Writeup

Emoji Cipher 
Flag: 
OPSWAT{3m0j15_4r3_jU5t_4n07h3r_3nc0d1ng_l4y3r_bu7_p4773rn5_4lw4y5_l34k_7f92a1
c84e6b39d5}  
 
 
Approach (Step by Step):  
The challenge contains a text file made up of emojis. At first, the symbols appear 
meaningless, but the clue file provides a direct mapping between emojis and hexadecimal 
digits. 
After converting the emojis into hexadecimal, the resulting bytes are XOR-encrypted with a 
repeating four-byte key. The known flag format provides the first part of the plaintext, which 
allows the XOR key to be recovered. 
The recovered key is: 
MOON 
Decrypting the full ciphertext produces the flag. 
1. Analyze the Clue File 
The clue file defines the following emoji-to-hexadecimal mapping: 
Emoji 
Hex digit 
  
0 
  
1 
  
2 


========== PAGE 33 ==========

 
 
 
  
3 
  
4 
  
5 
  
6 
  
7 
  
8 
  
9 
  
A 
  
B 
  
C 
  
D 
  
E 
  
F 
The variation selector character used by some emojis, FE0F, must be ignored during parsing. 
The ciphertext is therefore converted by: 
1. Removing spaces and line breaks. 
2. Removing variation selectors. 
3. Replacing each emoji with its hexadecimal digit. 
4. Grouping the result into pairs of hex digits. 
5. Converting each pair into one byte. 
The resulting ciphertext is: 
021f1c190c1b347d207f257f78107b3c7e10251b783b107a237f78267e3d107d232c7f2a
7c212811217b367d3f102d3b7a103f7a7a787c3c237a107a21387b377810237d79241079
2b767d2f7c2c777a28792d7d742b7a33 
2. Identify the Encryption Method 
The converted data is not readable text. Since the challenge is a cipher and the expected flag 
format is known, a repeating-key XOR is a likely possibility. 
The beginning of every flag is known: 
OPSWAT{ 
The first seven ciphertext bytes are: 
02 1f 1c 19 0c 1b 34 


========== PAGE 34 ==========

 
 
 
XORing these bytes with the known plaintext gives the first seven key bytes: 
Ciphertext: 02 1f 1c 19 0c 1b 34 
Plaintext:  4f 50 53 57 41 54 7b 
XOR result:  4d 4f 4f 4e 4d 4f 4f 
The result begins: 
MOONMOO 
This shows that the key repeats every four bytes: 
MOON 
The key is case-sensitive and must remain uppercase. 
3. Decrypt the Ciphertext 
Each ciphertext byte is XORed with the corresponding byte of the repeating key: 
plaintext[i] = ciphertext[i] ^ key[i % 4] 
The recovered plaintext is: 
OPSWAT{3m0j15_4r3_jU5t_4n07h3r_3nc0d1ng_l4y3r_bu7_p4773rn5_4lw4y5_l34k_7
f92a1c84e6b39d5} 
The capital U in jU5t is part of the flag and must be preserved. 
4. Reproduction Script 
from pathlib import Pathemoji_to_hex = {    " ": "0",    " ": "1",    
" ": "2",    " ": "3",    " ": "4",    " ": "5",    " ": "6",    
" ": "7",    " ": "8",    " ": "9",    " ": "a",    " ": "b",    
" ": "c",    " ": "d",    " ": "e",    " ": "f",}# Read the emoji 
ciphertext.text = Path("challenge.txt").read_text(encoding="utf-8")# 
Remove whitespace and variation selectors.text = 
"".join(text.split())text = text.replace("\ufe0f", "")# Remove the 
variation selector from the mapping keys as well.clean_mapping = {    
emoji.replace("\ufe0f", ""): value    for emoji, value in 
emoji_to_hex.items()}# Convert emojis to hexadecimal.hex_data = "" 
The output is: 


========== PAGE 35 ==========

 
 
 
OPSWAT{3m0j15_4r3_jU5t_4n07h3r_3nc0d1ng_l4y3r_bu7_p4773rn5_4lw4y5_l34k_7
f92a1c84e6b39d5} 
Conclusion 
The emojis represented hexadecimal digits. After converting them to bytes, the data was 
decrypted with a repeating XOR key. 
The known prefix OPSWAT{ revealed the repeating key: 
MOON
