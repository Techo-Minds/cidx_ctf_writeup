# Challenge 13: Where It All Started

## Flag
```
OPSWAT{Y0u_r34ch3d_3v3n_h3r3}
```

## Writeup

Where It All Started 
 
Flag: OPSWAT{Y0u_r34ch3d_3v3n_h3r3} 
 
 
Approach (Step by Step):  
Our Tech Partners started to teach people about cybersecurity. But it all started some years 
back. 
You need to find where it all started and you might even get what you're looking for. It's not 
always about the videos though. Where do you go when you wanna learn something. 
Overview 
The challenge was an OSINT investigation built around the event's Technology Partner. The 
clue pointed toward the partner's educational presence, and the phrase “where it all started” 
suggested looking for its oldest available content. 
The important twist was that the answer was not in the video itself; it was in the comments 
section of an early upload. 
Step 1 — Identify the Technology Partner 
We began with the CTF platform at opswatcidx.ctfd.io, where the landing page listed the 
event's partners: 
Label 
Organization 
Technology Partner 
OPSWAT Academy 


========== PAGE 42 ==========

 
 
 
Event Organizer 
CIDX 
Co-Hosted By 
Safeskillz 
Supported By 
Cyber Secured India 
The exact wording “Technology Partner” was the key. 
Rather than looking through OPSWAT's commercial technology partners, we needed to 
follow OPSWAT Academy, its cybersecurity education and certification initiative. 
Initial dead end: OPSWAT's general technology-partner directory contains product and 
security vendors, but this challenge's wording referred to the partner named on the CTF event 
page, not that directory. 
Step 2 — Interpret the Educational Clue 
The prompt included two helpful lines: 
Where do you go when you wanna learn something? 
It's not always about the videos though. 
Together, they suggested OPSWAT Academy's educational channels and resources. We 
investigated its official website and YouTube channel. 
The second line warned us not to focus only on the video footage. Descriptions, comments, 
and other surrounding metadata were worth checking. 
Step 3 — Find the Oldest Available Upload 
The phrase “where it all started” suggested inspecting the channel's earliest visible upload. 
We enumerated the channel's Videos tab using yt-dlp: 
yt-dlp --flat-playlist \ 
  --print '%(id)s | %(title)s' \ 
  'https://www.youtube.com/@OPSWATAcademy/videos' | tail -15 
From the investigated listing, the oldest entry was: 
yth7HfNUimY | Critical Infrastructure Cybersecurity Updates – June 2023 
Video details: 
• 
Title: Critical Infrastructure Cybersecurity Updates – June 2023 
• 
Upload Date: July 11, 2023 


========== PAGE 43 ==========

 
 
 
• 
Video ID: yth7HfNUimY 
• 
URL: https://www.youtube.com/watch?v=yth7HfNUimY 
The upload used CIP Cyber branding, an earlier identity associated with the Academy's 
online presence. 
This made it a plausible match for the challenge's “where it all started” hint. 
Note: “Oldest” refers to the oldest entry in the investigated channel listing, not necessarily 
every video the organization has ever produced. 
Step 4 — Extract the Flag from Video Comments 
The flag was not embedded in the footage. We inspected the video's comments instead. 
First, collect the video's metadata and comments: 
yt-dlp --skip-download --write-comments --dump-single-json \ 
  'https://www.youtube.com/watch?v=yth7HfNUimY' > video.json 
Next, extract strings matching the flag format: 
grep -oE 'OPSWAT\{[^}]+\}' video.json | sort -u 
Output: 
OPSWAT{Y0u_r34ch3d_3v3n_h3r3} 
The reported flag-bearing comment had the following metadata: 
Field 
Value 
Comment Author 
@akashmishra7334 
Comment Date 
October 18, 2026 
Comment Text 
OPSWAT{Y0u_r34ch3d_3v3n_h3r3} 
The comment date coincided with the CTF, consistent with a deliberately placed challenge 
flag. 
Tools Used 
• 
Browser / Page Inspection — CTF event page, partner labels, and Academy 
resources 
• 
yt-dlp — YouTube upload enumeration and comment retrieval 
• 
grep — Flag-pattern extraction 


========== PAGE 44 ==========
