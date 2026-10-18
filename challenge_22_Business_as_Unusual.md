# Challenge 22: Business as Unusual

## Flag
```
OPSWAT{n3g4t1v3_tr4nsf3r_c0up0n_st4ck1ng_1nfl4t3d_r3fund}
```

## Writeup

Business as Unusual 
Flag: OPSWAT{n3g4t1v3_tr4nsf3r_c0up0n_st4ck1ng_1nfl4t3d_r3fund} 
Approach (Step by Step): 
1. Challenge Architecture & Multi-Application Scope: The challenge presents three commercial 
web applications—Alpha (SecureTrust Bank), Beta (ByteMart), and Gamma (NovaCart)—each 
guarding one fragment of the flag through business logic vulnerabilities. 
2. App Alpha (SecureTrust Bank) — Negative Transfer Logic Flaw: The objective is unlocking the 
'Elite Wealth Circle' scheme requiring a $100,000 balance (starting from $750). Inspecting the transfer 
endpoint 'POST /alpha/api/transfer' revealed that while the system validates non-zero amounts and 
disallows self-transfers, it fails to enforce positive values. Submitting a negative transfer (-$100,000) 
debited the recipient and credited the sender's balance, instantly unlocking the tier and yielding 
fragment 1: 'OPSWAT{n3g4t1v3_tr4nsf3r_'. 
3. App Beta (ByteMart) — Unbounded Coupon Stacking: ByteMart offers a $10,000 flagship item 
with $200 user credit and a 10% coupon ('SAVE10'). The coupon application logic fails to check 
whether a promotion has already been applied to the current cart. Repeatedly submitting the coupon 
code compounded the discount exponentially until the order total fell below $200, allowing checkout 
and yielding fragment 2: 'c0up0n_st4ck1ng_'. 
4. App Gamma (NovaCart) — Inflated Refund Quantity: NovaCart requires acquiring an exclusive 
VIP voucher. The platform allows purchasing low-cost items and requesting refunds via 'POST 
/gamma/api/refund'. The refund handler accepts an arbitrary quantity without verifying against the 
original order quantity. Purchasing 1 unit of a $5 item and requesting a refund for 25,000 units credited 
$125,000 to the wallet, enabling the voucher purchase and yielding fragment 3: '1nfl4t3d_r3fund}'. 
5. Flag Assembly: Concatenating the three verified fragments in sequence produces the complete 
flag: OPSWAT{n3g4t1v3_tr4nsf3r_c0up0n_st4ck1ng_1nfl4t3d_r3fund}. 
 
 
 
 
 


========== PAGE 68 ==========
