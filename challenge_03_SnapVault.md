# Challenge 3: SnapVault

## Flag
```
OPSWAT{android_different_engineering_maybe_reverse_was}
```

## Writeup

SnapVault 
Flag:  
OPSWAT{android_different_engineering_maybe_reverse_was} 
 
 
Approach (Step by Step):  
Idea 
Six ordinary words were represented by SHA-256 digests scattered across the APK. The final 
digest was in the APK signing certificate, which an ordinary ZIP-member scan did not cover. 
The challenge hint required six fragments, sorted alphabetically, with lowercase content 
inside OPSWAT{...}. 
Step 1 — Inspect the APK's different storage surfaces 
An APK is a ZIP container, but its contents extend beyond app classes. The investigation 
covered the manifest, compiled resources, media metadata, SQLite database, native 
libraries, and signing certificate. 
Basic extraction: 
unzip SnapVault.apk -d snapvault 
Relevant files included: 
AndroidManifest.xml 
resources.arsc 
assets/db/app.db 
assets/media/sunset_beach.jpg 
lib/<ABI>/libnative-lib.so 
The manifest and resources.arsc need Android-aware parsing to recover named values 
reliably. Native strings can be inspected with strings, and image metadata with ExifTool. 


========== PAGE 9 ==========

 
 
 
Step 2 — Recognize hashes as possible fragments 
Several 64-character hexadecimal strings looked like routine integrity or build metadata. Five 
were recovered as SHA-256 hashes of common lowercase words. Rather than dismissing 
them as decoys, the six-part hint made the remaining surfaces worth checking. 
Word 
Location 
SHA-256 
androi
d 
Manifest 
BUILD_FINGERPRINT 
metadata 
f60ed56a9c8275894022fe5a7a1625c33bdb55
b729bb4e38962af4d1613eda25 
differ
ent 
EXIF metadata in 
assets/media/suns
et_beach.jpg 
9d6f965ac832e40a5df6c06afe983e3b449c07
b843ff51ce76204de05c690d11 
engine
ering 
resources.arsc, 
cache_partition_i
d 
8c85b6639e62e10b8330a8b883004b43c11905
3d7d62b512a98c77eb069627a7 
maybe 
SQLite 
cache_integrity, 
gallery_index row 
dcfff5eb40423f055a4cd0a8d7ed39ff6cb981
6868f5766b4088b9e9906961b9 
revers
e 
APK v2 signing 
certificate, subject 
common name 
b2d7f24e833051d5fc296d4a747281e9d155ec
fb636b983cfd70b51ed9b45a32 
was 
libnative-lib.so, 
checksumHint 
b63b99f6383ba713b57ddfc77737c5f71639fa
6f4663efe60a66a8c6d114b545 
For example, inspect the database with: 
sqlite3 snapvault/assets/db/app.db \ 
  'SELECT cache_bucket, checksum_algo, checksum_value FROM 
cache_integrity;' 
The relevant row identifies gallery_index, sha256, and the digest of maybe. 
SHA-256 was not decrypted. Candidate words were hashed and compared with the stored 
digests. A dictionary recovery loop is sufficient: 
import hashlib 
 
targets = { 
    "f60ed56a9c8275894022fe5a7a1625c33bdb55b729bb4e38962af4d1613eda25", 
    "9d6f965ac832e40a5df6c06afe983e3b449c07b843ff51ce76204de05c690d11", 
    "8c85b6639e62e10b8330a8b883004b43c119053d7d62b512a98c77eb069627a7", 
    "dcfff5eb40423f055a4cd0a8d7ed39ff6cb9816868f5766b4088b9e9906961b9", 
    "b2d7f24e833051d5fc296d4a747281e9d155ecfb636b983cfd70b51ed9b45a32", 


========== PAGE 10 ==========

 
 
 
    "b63b99f6383ba713b57ddfc77737c5f71639fa6f4663efe60a66a8c6d114b545", 
} 
 
with open("words.txt", "rb") as dictionary: 
    for line in dictionary: 
        word = line.rstrip(b"\r\n") 
        digest = hashlib.sha256(word).hexdigest() 
        if digest in targets: 
            print(word.decode(errors="replace"), digest) 
Step 3 — Recover the overlooked certificate fragment 
The sixth useful digest was the certificate subject's common name. The APK v2 signing block 
sits outside the ordinary ZIP entries, so extracting and scanning ZIP members alone missed it. 
This script extracts the certificate CN using Androguard and Cryptography: 
from loguru import logger 
from androguard.core.apk import APK 
from cryptography import x509 
from cryptography.x509.oid import NameOID 
 
logger.disable("androguard") 
apk = APK("SnapVault.apk") 
 
for certificate_der in apk.get_certificates_der_v2(): 
    certificate = x509.load_der_x509_certificate(certificate_der) 
    for attribute in certificate.subject.get_attributes_for_oid( 
        NameOID.COMMON_NAME 
    ): 
        print(attribute.value) 
The relevant CN was the hash of reverse. 
Step 4 — Verify and sort all six words 
Each recovered word matched its SHA-256 digest. Alphabetical sorting gave: 
android 
different 
engineering 
maybe 
reverse 
was 


========== PAGE 11 ==========

 
 
 
Join with underscores and preserve the uppercase wrapper: 
OPSWAT{android_different_engineering_maybe_reverse_was} 
What mattered 
The APK included distracting hashes and a conspicuous submit-this-looking message in a 
receipt image/resources. The complete solution came from six verified word-to-hash 
matches, including the signing certificate. No emulator was required for this recovery. 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 


========== PAGE 12 ==========
