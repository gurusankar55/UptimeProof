# UptimeProof — அடுத்த வாரம் செய்ய வேண்டியவை (ஒவ்வொரு step-ஆ)

ஒரு தடவைக்கு ஒரு step மட்டும் செய்யுங்க. முடிஞ்ச பிறகுதான் அடுத்த step-க்கு போங்க.

---

## STEP 1 — Folder ரெடி பண்ணுங்க
1. `UptimeProof.zip`-ஐ download பண்ணுங்க.
2. அதை right-click பண்ணி **Extract All**-ஐ கொடுங்க. Location-ல `D:\` கொடுங்க.
3. கடைசியா `D:\UptimeProof` folder-க்குள்ள `index.html`, `config.js`, `README.md`, `contracts`, `demo` இருக்கணும்.

---

## STEP 2 — GitHub-க்கு push பண்ணுங்க (Studio test-க்கு முன்னாடியே இதை செய்யணும்)
Demo status pages GitHub-ல இருந்தாதான் validators அதை படிக்க முடியும்.

PowerShell-ஐ open பண்ணி இந்த ஒரே command-ஐ paste பண்ணுங்க:

```
cd D:\UptimeProof; git init; git add .; git commit -m "UptimeProof v1"; git branch -M main; gh repo create uptimeproof --public --source . --push
```

`gh` வேலை செய்யலைன்னா இப்படி பண்ணுங்க: github.com-ல **uptimeproof**ங்கிற பேர்ல ஒரு public repo create பண்ணுங்க (README tick பண்ண வேண்டாம்). அப்புறம் இந்த command-ஐ ஓட்டுங்க:

```
cd D:\UptimeProof; git remote add origin https://github.com/gurusankar55/uptimeproof.git; git push -u origin main
```

**Check:** Browser-ல இந்த link-ஐ open பண்ணுங்க. JSON text தெரிஞ்சா சரி:
https://raw.githubusercontent.com/gurusankar55/uptimeproof/main/demo/status-ok.json

---

## STEP 3 — Studio-ல 2 contracts deploy பண்ணுங்க
1. studio.genlayer.com-க்கு போங்க. New contract create பண்ணி, `D:\UptimeProof\contracts\uptime_proof.py`-ல இருக்கிற code முழுசையும் paste பண்ணுங்க. முதல் 3 வரி ஏற்கனவே சரியான header-தான், அதை மாத்த வேண்டாம்.
2. **Demo contract:** `demo_mode`-ஐ **true** (tick) பண்ணி Deploy பண்ணுங்க. வர்ற address-ஐ Notepad-ல "DEMO"ன்னு எழுதி வச்சுக்கோங்க.
3. **Live contract:** அதே code-ஐ இன்னொரு தடவை `demo_mode` **false** வச்சு Deploy பண்ணுங்க. இந்த address-ஐ "LIVE"ன்னு எழுதி வச்சுக்கோங்க.

---

## STEP 4 — Studio test (DEMO contract-ல மட்டும்). ஒவ்வொரு test-க்கும் screenshot எடுங்க
Provider இடத்துல உங்க Studio account address-ஐயே போடுங்க (demo-ல அது allowed).
Value field-ல எப்பவும் முழு எண் மட்டும் போடுங்க (10, 2 மாதிரி).

**Test A — Tier 0 (outage இல்லை)**
- `create_plan`: provider = உங்க address, service_name = `NodeNova RPC`, component = `Ethereum Mainnet RPC`, status_url = `https://raw.githubusercontent.com/gurusankar55/uptimeproof/main/demo/status-ok.json`, sla_bp = `9990`, period = `2`, Value = `10`
- `accept_plan`: plan_id = `1`, Value = `2`
- **4 நிமிஷம் wait பண்ணுங்க.** அப்புறம் `settle`: plan_id = `1`
- `get_plan(1)` → `"state": "SETTLED"`, `"tier": 0` வரணும்.

**Test B — Tier 1:** Test A மாதிரியே, ஆனா url-ல கடைசி பேரை `status-minor.json`-ஆ மாத்துங்க. plan_id = `2`. ரிசல்ட் → tier 1.

**Test C — Tier 2 (மறைச்ச outage-க்கு evidence):** url `status-ok.json`, plan_id = `3`. `accept_plan` பண்ணின உடனே `report_outage`: plan_id = `3`, evidence_url = `https://raw.githubusercontent.com/gurusankar55/uptimeproof/main/demo/monitor-report.json`, note = `Endpoint 503, not on status page`. 4 நிமிஷம் கழிச்சு settle → tier 2.

**Test D — Tier 3:** url `status-major.json`, plan_id = `4` → tier 3. Customer-க்கு fee + bond ரெண்டும் போகணும்.

**Test E — Page இல்லை (404):** url `status-missing.json`, plan_id = `5`. 4 நிமிஷம் கழிச்சு `settle` பண்ணுங்க, அப்புறம் **ஒவ்வொரு 1 நிமிஷத்துக்கும் ஒரு தடவை** `settle`-ஐ திரும்ப பண்ணுங்க. 4–5 தடவை ஆனதும் `get_plan(5)` → `"state": "NO_DATA_REFUND"`.
(முதல் சில settle-ல `UNREACHABLE_RETRY_LATER` வரும். அது சரிதான்.)

**Test F — Cancel:** `create_plan` (url status-ok, Value 5) → plan_id = `6`. Accept பண்ண வேண்டாம். நேரா `cancel_unaccepted(6)` → `"state": "CANCELLED"`.

எல்லா screenshot-ஐயும் `D:\UptimeProof\evidence\` folder-ல போடுங்க. பேர் இப்படி வைங்க: `A_tier0.png`, `B_tier1.png`, `C_tier2.png`, `D_tier3.png`, `E_nodata.png`, `F_cancel.png`.

ஒரு settle "failed"/"undetermined"ன்னு வந்தா, 1 நிமிஷம் கழிச்சு அதே settle-ஐ திரும்ப பண்ணுங்க.

---

## STEP 5 — Address-களை site-ல போட்டு push பண்ணுங்க
இந்த command-ல `DEMO_ADDR`, `LIVE_ADDR`-ங்கிற இடத்துல உங்க ரெண்டு address-ஐயும் போட்டு PowerShell-ல ஓட்டுங்க:

```
cd D:\UptimeProof; (Get-Content config.js) -replace 'PASTE_DEMO_ADDRESS','DEMO_ADDR' -replace 'PASTE_LIVE_ADDRESS','LIVE_ADDR' | Set-Content config.js; git add .; git commit -m "Add contract addresses and Studio test evidence"; git push
```

---

## STEP 6 — Vercel-ல site போடுங்க
1. vercel.com-க்கு போங்க (HireProof போட்ட அதே account). **Add New → Project** கொடுங்க.
2. `uptimeproof` repo-வுக்கு பக்கத்துல இருக்கிற **Import** கொடுங்க. Framework = **Other**. அப்புறம் **Deploy** கொடுங்க.
3. வர்ற link-ஐ (எ.கா. uptimeproof.vercel.app) எழுதி வச்சுக்கோங்க.
4. Site-ல **Connect wallet** பண்ணி, **Latest**-ஐ கொடுங்க. Plan #6 தெரிஞ்சா site சரியா வேலை செய்யுது.

---

## STEP 7 — Demo video (2–3 நிமிஷம்)
1. Site-ஐ ஓப்பன் பண்ணுங்க. Problem, How it works, SLA table மூணையும் scroll பண்ணி காட்டுங்க.
2. App-ல Demo page **"small outage → tier 1"**-ஐ select பண்ணுங்க. Provider இடத்துல உங்க address போட்டு, Fee 5 கொடுத்து **Lock fee & create plan** கொடுங்க.
3. **Accept as provider** கொடுங்க (bond 1). 4 நிமிஷம் கழிச்சு **Settle now** கொடுங்க. Result-ல Tier 1 தெரியறதை காட்டுங்க.
4. YouTube-ல **Unlisted**-ஆ upload பண்ணுங்க.

---

## STEP 8 — X post
`SUBMIT_TEXT.md`-ல இருக்கிற X thread-ஐ post பண்ணுங்க. உங்க video link அல்லது site screenshot-ஐ attach பண்ணுங்க.

---

## STEP 9 — Portal submit
`SUBMIT_TEXT.md`-ஐ open பண்ணுங்க. Links எல்லாத்தையும் fill பண்ணி, Portal-ல **Projects** category-ல submit பண்ணுங்க.

ஏதாவது error வந்தா, அந்த error-ஓட screenshot-ஐ free Claude chat-ல அனுப்பி கேளுங்க.
