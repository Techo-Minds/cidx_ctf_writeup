# Challenge 25: DotWave

## Flag
```
OPSWAT{24_12_2031_BLETCHLEY}
```

## Writeup

DotWave 
Flag: OPSWAT{24_12_2031_BLETCHLEY} 
Approach (Step by Step): 
1. Audio File Triage & Spectral Inspection: The challenge supplies three WAV audio files 
('_first_.wav', '_second_.wav', '_third_.wav'). Generating audio spectrograms with ffmpeg 
(showspectrumpic) revealed that files 2 and 3 contained steganographically drawn text and 
mathematical tokens, while file 1 was a spoken decoy. 
2. Spectrogram Extraction of Event Date: Inspecting the high-frequency spectrum of 
'_second_.wav' revealed rendered text: 'Football Match is fixed on 24/12/2031. Location will be 
conveyed later on.' Extracting the date gives '24_12_2031'. 
3. Spectral Token Extraction for Location: The spectrogram of '_third_.wav' contained an encoded 
location sequence: '.7 .003 .4 .05 .6 .1 .003 .4 .00 +9'. Extracting each token's non-zero digit d and 
decimal place p revealed a repeating structural pattern at positions 2-3 and 7-8. 
4. Algebraic Mapping & Location Resolution: Evaluating the sequence against geographical 
intelligence targets mapped the values to the 9-letter landmark 'BLETCHLEY' (referencing Bletchley 
Park, the historic codebreaking headquarters). 
5. Flag Assembly: Combining the extracted date and location within the challenge wrapper produced 
OPSWAT{24_12_2031_BLETCHLEY}. 
 
 
 
 
 
 
 
 
 
 
 


========== PAGE 71 ==========
