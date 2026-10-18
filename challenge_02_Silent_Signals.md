# Challenge 2: Silent Signals

## Flag
```
OPSWAT{w1llp0w3r_str0ng3r_th4n_th3_0bqu5c4710n}
```

## Writeup

Silent Signals 
Flag: OPSWAT{w1llp0w3r_str0ng3r_th4n_th3_0bqu5c4710n} 
 
 
Approach (Step by Step):  
1. Silent Signals 
Points: 300 
Author: Sapphire 
Techniques: packet analysis, DNS investigation, flag reconstruction 
Input: network_investigation.pcap 
Idea 
The capture contained a large amount of distracting traffic. The useful trail was a repeated 
DNS name, which led to three flag fragments stored in different DNS record types. 
Step 1 — Identify the useful DNS traffic 
The file was a classic PCAP containing 66,429 packets and measuring 20,973,362 bytes. 
Searching for obvious flag strings produced distractions rather than a complete answer. 
DNS analysis revealed 175 query/response pairs involving: 
telementry.ctf.safeskillz.co.uk 
Preserve the spelling telementry when searching the capture. A useful Wireshark display 
filter is: 
dns.qry.name contains "safeskillz" 
Equivalent extraction with TShark: 
tshark -r network_investigation.pcap -Y 'dns' \ 
  -T fields -e dns.qry.name | sort | uniq -c | sort -nr 
This exposed the parent domain: 


========== PAGE 6 ==========

 
 
 
ctf.safeskillz.co.uk 
Step 2 — Apply the three-part hint 
The challenge announcement stated that Silent Signals had three flag parts. The TXT record 
alone was therefore incomplete. 
Queries to the parent domain recovered these records during the investigation: 
Type 
Numeric 
type 
Relevant answer 
Flag contribution 
TXT 
16 
w1llp0w3r_str0ng3r 
w1llp0w3r_str0
ng3r 
HTTP
S 
65 
1 _th4n_th3. 
alpn=h2,h3,port=443 
_th4n_th3 
SVCB 
64 
1 _0bqu5c4710n. 
alpn=h2,h3,port=9443 
_0bqu5c4710n 
Google DNS and Cloudflare independently returned the three fragments. The ANY response 
from Google also exposed them together. 
Reproduction commands: 
dig @8.8.8.8 ctf.safeskillz.co.uk TXT +short 
dig @8.8.8.8 ctf.safeskillz.co.uk TYPE65 +short 
dig @8.8.8.8 ctf.safeskillz.co.uk TYPE64 +short 
Or use DNS over HTTPS: 
import json 
import urllib.request 
 
for record_type in (16, 65, 64): 
    url = ( 
        "https://dns.google/resolve?" 
        f"name=ctf.safeskillz.co.uk&type={record_type}" 
    ) 
    with urllib.request.urlopen(url, timeout=20) as response: 
        answer = json.load(response) 
    print(record_type, answer.get("Answer", [])) 
These were live DNS observations from the solve date. Later DNS changes may affect 
reproduction. 


========== PAGE 7 ==========

 
 
 
Step 3 — Reconstruct the flag 
Remove DNS presentation syntax, including the trailing root dot on the HTTPS/SVCB targets. 
Concatenate the three contributions in sentence order and add the flag wrapper. 
OPSWAT{w1llp0w3r_str0ng3r_th4n_th3_0bqu5c4710n} 
Character check: the last fragment contains q in 0bqu5c4710n. It must not be silently 
corrected to f.  
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 


========== PAGE 8 ==========
