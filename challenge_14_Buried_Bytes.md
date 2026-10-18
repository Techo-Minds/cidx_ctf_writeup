# Challenge 14: Buried Bytes

## Flag
```
OPSWAT{byt3s_r3m3mb3r_3v3ryth1ng}
```

## Writeup

Buried Bytes 
Flag: OPSWAT{byt3s_r3m3mb3r_3v3ryth1ng}  
 
 
Approach (Step by Step): Buried Bytes — Miscellaneous CTF Write-up 
Challenge: Buried Bytes 
Category: Miscellaneous 
Flag: OPSWAT{byt3s_r3m3mb3r_3v3ryth1ng} 
Challenge prompt 
A simple image hides a deeper mystery. Analyze the file, follow the clues, and uncover what 
lies beneath the surface. 
Solution overview 
The supplied challenge.jpg looks like an ordinary JPEG, but its data continues after the JPEG 
end-of-image marker (FF D9). The trailing bytes form a ZIP archive. That archive contains a 
small ELF program named relay; its read-only data section holds a transformed bitmap. 
Reversing the transform reconstructs a BMP whose visible text is the flag. 
The key idea is to inspect the file structure and the embedded program, rather than relying 
only on the visible image, EXIF metadata, or printable strings. 
1. Find the data appended after the JPEG 
Basic identification reports a 786×535 JFIF JPEG. A byte-level check finds the first FF D9 at 
offset 0x8E0E (36,366). The next bytes are PK\x03\x04, the ZIP local-file header signature. 
A quick check in Python: 
from pathlib import Path 
 


========== PAGE 45 ==========

 
 
 
image = Path("challenge.jpg").read_bytes() 
eoi = image.find(b"\xff\xd9") 
assert eoi >= 0 
trailer = image[eoi + 2:] 
assert trailer.startswith(b"PK\x03\x04") 
Path("buried.zip").write_bytes(trailer) 
 
The JPEG is 60,986 bytes; the appended ZIP is 24,618 bytes. Listing the archive shows one 
member, relay (663,208 bytes): 
unzip -l buried.zip 
unzip buried.zip 
file relay 
 
file relay identifies it as a 64-bit x86-64 ELF executable. 
2. Inspect the embedded program 
Running relay prints status clues such as R-17 :: 0x4D2F and RELAY STATE :: 
UNRESOLVED, but it does not print the flag or write the hidden image. Static inspection is more 
useful: 
nm -S relay | grep -E 'payload_fragment_bin|stage_frame' 
readelf -S relay | grep -A1 '\.rodata' 
objdump -d -Mintel relay 
 
The symbol table identifies: 
• 
payload_fragment_bin at virtual address 0x402040 
• 
payload length 0x9e376 bytes (648,054) 
• 
.rodata beginning at virtual address 0x402000, file offset 0x2000 
So the payload begins at ELF file offset 0x2040. 
The disassembly of stage_frame shows the transform. For every source byte at index i, it 
XORs the byte with 31*i + 0x5A (keeping the low byte) and writes it to the reversed 
destination index: 
frame[n - i - 1] = payload[i] XOR ((31*i + 0x5A) & 0xFF) 
 
The reversal and XOR are why the embedded data does not start with the normal BMP 
signature. 


========== PAGE 46 ==========

 
 
 
3. Decode the frame 
This complete Python script extracts the appended archive, reads the embedded payload, and 
reconstructs the BMP: 
from io import BytesIO 
from pathlib import Path 
from zipfile import ZipFile 
 
image = Path("challenge.jpg").read_bytes() 
eoi = image.find(b"\xff\xd9") 
if eoi < 0: 
    raise ValueError("JPEG EOI marker not found") 
 
zip_data = image[eoi + 2:] 
with ZipFile(BytesIO(zip_data)) as archive: 
    print("Archive members:", archive.namelist()) 
    elf = archive.read("relay") 
 
# Values obtained from nm/readelf on this challenge's ELF. 
payload_offset = 0x2040 
payload_length = 0x9E376 
payload = elf[payload_offset:payload_offset + payload_length] 
if len(payload) != payload_length: 
    raise ValueError("Embedded payload is truncated") 
 
frame = bytearray(payload_length) 
for i, byte in enumerate(payload): 
    frame[payload_length - i - 1] = byte ^ ((31 * i + 0x5A) & 0xFF) 
 
if frame[:2] != b"BM": 
    raise ValueError("Decoded bytes do not have a BMP header") 
Path("relay_stage1.bmp").write_bytes(frame) 
 
# Optional checksum matching the rolling calculation in main(). 
check = 0x4D2F 
for byte in frame: 
    check = ((check << 3) | (check >> 29)) & 0xFFFFFFFF 
    check ^= byte 
print(f"Decoded BMP: relay_stage1.bmp ({len(frame)} bytes)") 
print(f"Rolling check: 0x{check:08X}") 
 
The decoded file is a 900×240, 24-bit BMP. Convert it to PNG or open it directly: 


========== PAGE 47 ==========

 
 
 
magick relay_stage1.bmp relay_stage1.png 
 
The image displays: 
OPSWAT{byt3s_r3m3mb3r_3v3ryth1ng} 
 
Approaches considered 
• 
Visible image / ordinary strings: Neither reveals the flag; the flag is rendered as 
pixels in the recovered frame. 
• 
File metadata only: The useful clue is not a conventional EXIF comment but the ZIP 
archive after the JPEG EOI marker. 
• 
Archive inspection: unzip -l exposes the relay member. 
• 
Running the ELF: It prints status messages but does not dump the image. Inspecting 
its symbols and stage_frame disassembly reveals the decode operation. 
• 
Steganography tools: A pixel-plane or password-based extraction is unnecessary once 
the appended archive and ELF transform are identified. 
Why the flag fits 
The final text—“bytes remember everything”—describes the construction: data that looks like a 
simple image has additional bytes appended to it, and the hidden binary preserves a complete 
bitmap in an encoded form. 
  
 
 
 
 
 
 
 
 
 


========== PAGE 48 ==========
