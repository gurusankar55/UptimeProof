# UptimeProof — Portal submission (copy-paste)

## Links (fill these after Steps 3–7)
- GitHub: https://github.com/gurusankar55/uptimeproof
- Live site (Vercel): ______________________
- Demo contract (Studio): 0x______________________
- Live contract (Studio): 0x______________________
- Demo video (YouTube): ______________________
- X thread: ______________________
- Evidence screenshots: https://github.com/gurusankar55/uptimeproof/tree/main/evidence

## Title
UptimeProof

## Tagline
SLA escrow for RPC & API services: validators read the provider's status page and refund missed uptime automatically.

## Description (≈560 chars)
Crypto teams pay RPC/API providers for 99.9% uptime, but when the endpoint goes down the refund needs a support ticket and the provider decides if it counted. UptimeProof locks the fee in a per-plan escrow. After the period, GenLayer validators read the provider's own status page (plus an optional third-party evidence link), measure real outage on the bought component and agree on an SLA tier. Fixed code pays 0/25/50/100% back. No shared pool, separate demo and live contracts, refund if the page vanishes.

## Category / tags
Projects — DeFi Infrastructure / Dispute Resolution / Oracles

---

## X thread (post from your account)

1/
Your RPC provider promised 99.9% uptime.
It went down for 3 hours. Your bot missed trades.
Did you get your SLA refund? Most teams never do.

I built UptimeProof on @GenLayer 🧵

2/
How it works:
• Customer locks the monthly fee in escrow
• Provider accepts (and can post a bond)
• After the period, GenLayer validators read the provider's OWN status page
• Contract pays the refund by a fixed SLA table

3/
Why GenLayer?
Reading a status page is subjective: which incident hit YOUR component? Outage or just "degraded"? Planned maintenance?
Validators judge that and must agree on the same tier. The money split stays plain code.

4/
Status page hides the outage? Add one public monitor link as evidence.
Status page goes 404? After 3 reads + grace, the customer gets fee + bond back.

5/
Live demo + code 👇
Site: <your vercel link>
GitHub: github.com/gurusankar55/uptimeproof
