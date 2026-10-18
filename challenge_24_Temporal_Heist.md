# Challenge 24: Temporal Heist

## Flag
```
OPSWAT{3phim3r41_5y5t3m_Un10ck3d}
```

## Writeup

Temporal Heist 
Flag: OPSWAT{3phim3r41_5y5t3m_Un10ck3d} 
Approach (Step by Step): 
1. Session Architecture & Chain Analysis: Temporal Heist implements a multi-stage web 
challenge where the root page issues a session cookie ('sid') and provides an ephemeral, single-use 
endpoint. Each stage dynamically encodes the destination path for the subsequent stage, requiring 
automated session-consistent traversal. 
2. Automated Stage-by-Stage Decoding: Using a Python requests.Session() to maintain state, we 
automated the sequential decoding pipeline: Stage 1 ('/l/<token>') returned a hex-encoded path 
('/d/...'). Stage 2 ('/d/<token>') returned a reversed string with backslashes ('\esrever\...'), which when 
reversed and normalized yielded '/reverse/<uuid>'. Stage 3 provided the direct endpoint '/verify/<id>'. 
3. Header Verification Challenge: At '/verify/<id>', the response body is blank, but an 'X-HTTP-
Verification' response header supplies an arithmetic expression (e.g., '12 * 8'). Evaluating the 
expression and re-submitting the answer in an 'X-HTTP-Verification' request header to the same 
endpoint validated the check and unlocked the hunt phase ('/hunt/<uuid>'). 
4. Decoy Filtering & Flag Extraction: The '/hunt/<uuid>' endpoint returned a JSON list of 5 candidate 
URLs ('/c/<uuid>'). Querying the candidates within the same session identified the authentic endpoint 
(filtering out 4 decoy responses), returning the final flag payload: 
OPSWAT{3phim3r41_5y5t3m_Un10ck3d}. 
 
 
 
 
 
 
 
 
 
 
 


========== PAGE 70 ==========
