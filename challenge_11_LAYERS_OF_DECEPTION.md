# Challenge 11: LAYERS OF DECEPTION

## Flag
```
OPSWAT{tr4c3_th3_s1gn4l_cr4ck3d}
```

## Writeup

LAYERS OF DECEPTION 
Flag: OPSWAT{tr4c3_th3_s1gn4l_cr4ck3d} 
 
 
Approach (Step by Step):  
Layers of Deception — Write-up 
Category: Network Forensics 
Points: 300 
The challenge provided a PCAP containing uploads to a suspicious file-sharing server at 
203.0.113.50. The goal was to recover the exfiltrated file and identify the attacker. 
1. Examine the upload traffic 
I reviewed the HTTP traffic directed at 203.0.113.50:80. The uploads used 
multipart/form-data, so I reassembled the TCP streams and extracted every uploaded 
file. 
Most files were normal-looking documents and images. One upload stood out: 
capture_0917.pcap 


========== PAGE 39 ==========

 
 
 
This was another packet capture hidden among the uploaded files. 
2. Inspect the nested capture 
The recovered PCAP contained TLS traffic. Since TLS certificates are transmitted during the 
handshake before encryption begins, I extracted the certificates from each Server Hello / 
Certificate message and inspected their subject fields. 
Most certificate subjects looked ordinary. One certificate was clearly suspicious: 
CN = vault.corp-secure.example 
O  = Corp Secure Ltd 
OU = Jy:lSP_k:kEr 
L  = JP1kc}5&yVs?*fAPE1 
The unusual OU and L values were not normal certificate metadata. 
3. Decode the hidden values 
The strings use Base92 encoding. Decoding them separately gives: 
Jy:lSP_k:kEr         → tr4c3_th3 
JP1kc}5&yVs?*fAPE1   → s1gn4l_cr4ck3d 
Combining both parts with the required underscore produces the flag. 
Flag 
OPSWAT{tr4c3_th3_s1gn4l_cr4ck3d} 
 
 
 
 
 
 
 
 


========== PAGE 40 ==========
