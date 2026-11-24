# UptimeProof — SLA escrow for RPC & API services on GenLayer

**Your RPC promised 99.9%. Now the contract checks.**

Crypto teams pay RPC / API / node providers for an uptime promise (SLA). When the endpoint goes down, the refund ("SLA credit") only comes if you open a ticket and argue, and the provider decides whether the outage counted. Most teams never claim it.

UptimeProof turns the provider's **own public status page** into an automatic refund:

1. **Customer creates a plan.** It names the provider wallet, the component bought (e.g. `Ethereum Mainnet RPC`), the status page URL, the SLA % and the period. The fee is locked in **this plan's own escrow**.
2. **Provider accepts.** The provider can also post a bond. That starts the period. If the provider doesn't accept, the customer cancels and gets 100% back.
3. **Customer can report a hidden outage.** If the status page under-reports, the customer adds one public evidence link (e.g. a third-party uptime monitor).
4. **Anyone settles after the period.** GenLayer validators read the status page (and the evidence), measure real outage on that component inside the window, and must agree on the same **SLA tier**. The contract pays both sides in the same transaction.

## SLA credit table (fixed in code)

| Tier | Uptime vs SLA | Customer refund | Provider gets |
|---|---|---|---|
| 0 | SLA met | 0 | 100% fee + bond back |
| 1 | missed by < 1.00% | 25% of fee | 75% fee + bond |
| 2 | missed by < 5.00% | 50% of fee | 50% fee + bond |
| 3 | worse | 100% fee + bond | 0 |
| — | status page unreadable after 3 reads + grace | 100% fee + bond | 0 |

## Why the money logic is safe

- **The AI only measures downtime.** Each validator turns its reading into a tier with the same code. Consensus is `strict_eq` on the tier. The payout split is plain integer math.
- **No shared pool.** Each plan holds only its own fee and bond.
- **Demo and live are separate deployments.** `demo_mode=true` counts the period in minutes and uses the demo status pages in `/demo`. `demo_mode=false` counts days, and the provider must be a different wallet.
- **Lifecycle guards.** Only the provider can accept, before `accept_by`. Only the customer can cancel (unaccepted only) or add evidence (once, during the period + report window). You can only settle after period + report window, with a retry gap between status reads. Settled plans are final.
- **Status page is the provider's duty.** If it is 404 / login-walled / removed for 3 reads and the grace period passes, the customer is refunded fee + bond.
- **Prompt-injection aware.** Validators are told page and evidence text is data, never instructions.
- **Real outages only.** Announced maintenance and "degraded performance" with succeeding requests are not counted. Incidents are clipped to the window and overlaps aren't double counted.

| Window | Demo | Live |
|---|---|---|
| Provider accept window | 30 min | 7 days |
| Report window after period | 1 min | 2 days |
| Retry gap between reads | 30 s | 6 h |
| Grace before no-data refund | 3 min | 7 days |

## Contract API

| Method | Who | Notes |
|---|---|---|
| `create_plan(provider, service_name, component, status_url, sla_bp, period)` payable | customer | `sla_bp` 9000–9999 (9990 = 99.90%). The value sent is the fee. |
| `accept_plan(plan_id)` payable | provider | The value sent is the optional bond. |
| `cancel_unaccepted(plan_id)` | customer | Only while the plan is OPEN. |
| `report_outage(plan_id, evidence_url, note)` | customer | Once per plan. |
| `settle(plan_id)` | anyone | Returns `SETTLED_TIER_n`, `UNREACHABLE_RETRY_LATER` or `NO_DATA_REFUND`. |
| `get_plan`, `get_count`, `list_plans`, `get_info` | view | JSON strings |

## Demo status pages (`/demo`)

| File | Meaning | Expected with SLA 99.90% |
|---|---|---|
| `status-ok.json` | no outage | Tier 0 |
| `status-minor.json` | 0.5% outage on the component, plus one dashboard incident that doesn't count | Tier 1 |
| `status-ok.json` + evidence `monitor-report.json` | 3% outage hidden from the status page | Tier 2 |
| `status-major.json` | 20% outage | Tier 3 |
| `status-missing.json` (does not exist → 404) | page removed | NO_DATA_REFUND after 3+ reads |

## Files

- `contracts/uptime_proof.py` — the Intelligent Contract
- `index.html`, `config.js` — the dApp (static, uses genlayer-js on GenLayer Studio)
- `demo/` — demo status pages and the monitor report
- `evidence/` — Studio test screenshots

Built by Sankar D (gurusankar55) for the GenLayer Builders Portal.
