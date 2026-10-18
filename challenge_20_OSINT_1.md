# Challenge 20: OSINT-1

## Flag
```
OPSWAT{punnamada_steel_footbridge}
```

## Writeup

OSINT-1 
Flag: OPSWAT{punnamada_steel_footbridge} 
Approach (Step by Step): 
1. Artifact & Metadata Triage: The challenge provides a high-resolution photograph 'OSINT-1.jpg'. 
Inspecting the EXIF metadata using PIL / exiftool revealed the capture device as a Motorola edge 60 pro 
with image dimensions 3648x2736 and capture timestamp 2026:08:29 13:55:05. While GPS 
coordinates were stripped, the high resolution preserved vital architectural and environmental details. 
 
Figure 1: Provided Challenge Image (OSINT-1.jpg) 
2. Visual & Cultural Reconnaissance: Examining the visual landscape, multiple traditional wooden 
houseboats with distinctive curved thatched bamboo roofs (known as Kettuvallam) are moored along 
both embankments of a placid inland waterway. Kettuvallam houseboats are iconic and exclusive to 
the Kerala Backwaters in southwestern India. Dense rows of coconut palms line narrow 
embankments, pinpointing the geographical location to Alappuzha (Alleppey), Kerala. 
3. Vantage Point & Camera Perspective Analysis: The camera perspective is positioned at an 
elevated height centered directly above the water channel, looking straight down the canal towards a 


========== PAGE 66 ==========

 
 
 
broader lake basin. The height and centered alignment confirm that the photographer was standing on 
a pedestrian footbridge spanning across the canal. 
4. Canal Network Mapping to Punnamada Lake: Tracing the canal infrastructure of Alappuzha 
connecting into Punnamada Lake (famous for the Nehru Trophy Boat Race), we mapped the exact 
navigable waterway known as the East Thottathodu canal. The orientation of the lake mouth and 
shoreline profile narrows the vantage point to the pedestrian crossing over this specific channel. 
5. Geolocation & Landmark Identification: Using satellite imagery and street photography along 
the East Thottathodu canal leading into Punnamada Lake, we pinpointed the exact steel footbridge 
spanning the waterway at coordinates 9°30'43.9"N 76°21'15.3"E (9.512201, 76.354246). The official 
landmark name on Google Maps is 'Punnamada Steel Footbridge'. 
 
Figure 2: Ground-Level View of Punnamada Steel Footbridge 
6. Visual Cross-Verification: Cross-referencing ground-level photospheres taken from the bridge 
deck confirmed an exact 1:1 match: the canal curvature, concrete embankment mooring poles, and 
resort tree line align identically with OSINT-1.jpg. 
7. Flag Formulation: Converting the confirmed landmark name into lowercase snake_case enclosed 
within the required wrapper yields OPSWAT{punnamada_steel_footbridge}.
