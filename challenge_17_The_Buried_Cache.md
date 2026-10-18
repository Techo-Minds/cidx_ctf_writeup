# Challenge 17: The Buried Cache

## Flag
```
OPSWAT{S7ruggl3_f0r_7h3_fl4g???}
```

## Writeup

The Buried Cache 
Flag: OPSWAT{S7ruggl3_f0r_7h3_fl4g???} 
 
 
Approach (Step by Step):  
Event: OPSWAT CIDX CTF 2026 
Category: Digital Forensics 
Author: NotAnHacker 
Challenge File: website_challenge.zip 
Flag: OPSWAT{S7ruggl3_f0r_7h3_fl4g???} 
Challenge Description 
A developer working on a Survival Server website accidentally committed a sensitive file to a 
Git repository. After realizing the mistake, they deleted the file and made more than 100 
additional commits to hide the evidence. 
Our task was to investigate the repository history, recover the deleted file, and find the 
hidden flag. 
Solution 
We were provided with a ZIP archive containing a website project and its complete Git 
history. 
The repository contained 153 commits, most of which were small CSS changes. Instead of 
manually checking every commit, I focused on finding deleted files in the repository history. 


========== PAGE 57 ==========

 
 
 
Step 1 — Extracting the Repository 
First, I extracted the archive and navigated to the project directory. 
unzip website_challenge.zip 
cd server_website 
I checked the repository status and commit history. 
git status 
git log --oneline 
git log --oneline | wc -l 
The repository had 153 commits. 
Most commits contained minor styling changes, which appeared to be distractions. 
Step 2 — Finding Deleted Files 
Since the challenge mentioned that the developer deleted a sensitive file, I searched 
specifically for deletion commits. 
git log --diff-filter=D --summary --oneline 
This returned: 
bb1869a Removed corrupted cache blob, breaking the build 
 delete mode 100644 temp_cache.json 
This was the most interesting finding. 
The commit deleted a file named temp_cache.json, and the commit message described it 
as a corrupted cache blob. 
Instead of trusting the commit message, I decided to inspect the deleted file. 
Step 3 — Recovering the Deleted File 
Git keeps previous versions of files in its history, even after they are deleted from the latest 
commit. 
I recovered the file from the parent of the deletion commit: 
git show bb1869a^:temp_cache.json > temp_cache.json 


========== PAGE 58 ==========

 
 
 
The recovered file was approximately 109 KB. 
Opening it revealed a JSON structure containing a cache_blob field. 
{ 
  "status": "offline", 
  "cache_blob": "..." 
} 
The cache_blob value was a long Base64-encoded string. 
This suggested that the file contained more than ordinary server-status information. 
Step 4 — Decoding the Cache Blob 
I wrote a short Python script to extract and decode the Base64 data. 
import json 
import base64 
 
with open("temp_cache.json", "r") as f: 
    data = json.load(f) 
 
decoded = base64.b64decode(data["cache_blob"]) 
 
with open("cache_blob.bin", "wb") as f: 
    f.write(decoded) 
The decoded output was approximately 82 KB. 
I then checked its actual file type: 
file cache_blob.bin 
Output: 
cache_blob.bin: JPEG image data, JFIF standard 1.01, 640x640 
The supposed cache data was actually a JPEG image. 
Step 5 — Recovering the Flag 
I renamed the decoded file to make it easier to open: 
mv cache_blob.bin recovered.jpg 


========== PAGE 59 ==========

 
 
 
Opening the image revealed a neon-purple wolf logo. 
The flag was printed directly in the bottom-right corner of the image. 
No additional decoding or steganography techniques were required. 
Final Flag 
OPSWAT{S7ruggl3_f0r_7h3_fl4g???} 
Conclusion 
This challenge focused on Git forensics and recovering deleted information from repository 
history. 
The main difficulty was the large number of commits added to distract from the deleted file. 
By filtering the Git history for deletions, I was able to identify the relevant commit without 
manually reviewing all 153 commits. 
The recovered JSON file contained a Base64-encoded JPEG, which displayed the flag directly. 
The challenge demonstrated that deleting sensitive information from a Git repository does 
not automatically remove it from the repository's history. 
Tools Used: Git, Python, unzip, file, Base64 decoding. 
Solving Path: 
ZIP Archive → Git History → Deleted JSON → Base64 Decode → JPEG → Flag 
 
 
 
 
 
 
 


========== PAGE 60 ==========
