# Estimation Template — JSON Output Format
## Version 6.0 FINAL — Updated 2026-02-07

---

## Critical Rules

1. **total_score** is ALWAYS a percentage (0-100), never raw points
2. **ALL criteria** must appear in detailed_scoring, even excluded ones (0/0)
3. **offer_opening** is ALWAYS generated, including for BLOCKED vessels
4. Draft restrictions → P5, never P8. Age → never triggers BLOCK.
5. Use field `draft` for draught ratio (not `max-draught`)

---

## Example 1: Good match (no blocks)

```json
{
  "vessel_name": "MV EXAMPLE",
  "total_score": 81.1,
  "total_score_raw": 73,
  "dynamic_max_score": 90,
  "confidence": 85,
  "confidence_note": "High confidence. P2, P3, P6, P8 excluded (no data).",
  "recommendation": "STRONG",
  "short_summary": "Excellent match — well positioned, good timing, cargo fits.",
  
  "relationship_warning": { "status": "CLEAR" },
  
  "offer_opening": {
    "intent": "attract",
    "email_text": "Pls find below cargo which suits mv Example nicely — she's well positioned and timing works. Wld appreciate yr freight ideas.",
    "messenger_text": "have a nice cargo for Example — close by, good timing. pls see below, lmk?",
    "key_selling_points": ["proximity", "timing", "cargo fit"],
    "key_concerns": [],
    "questions_to_ask": []
  },
  
  "detailed_scoring": {
    "proximity":              { "code": "P1",  "score": 14.0, "max_score": 20, "decision": "accepted" },
    "regional_patterns":      { "code": "P1A", "score": 12,   "max_score": 15, "decision": "accepted" },
    "owner_preferences":      { "code": "P2",  "score": 0,    "max_score": 0,  "decision": "excluded" },
    "port_accessibility":     { "code": "P2A", "score": 18,   "max_score": 20, "decision": "accepted" },
    "cargo_preferences":      { "code": "P3",  "score": 0,    "max_score": 0,  "decision": "excluded" },
    "last_ports":             { "code": "P4",  "score": 7,    "max_score": 10, "decision": "accepted" },
    "intake_capacity":        { "code": "P5",  "score": 12,   "max_score": 15, "decision": "accepted" },
    "openarea_modifier":      { "code": "P6",  "score": 0,    "max_score": 0,  "decision": "excluded" },
    "readiness_eta":          { "code": "P7",  "score": 10,   "max_score": 10, "decision": "accepted" },
    "restrictions_compliance": { "code": "P8", "score": 0,    "max_score": 0,  "decision": "excluded" }
  }
}
```

Score: active = P1+P1A+P2A+P4+P5+P7 = 90. Raw = 73. Pct = 81.1%.

---

## Example 2: BLOCKED vessel — offer_opening IS generated

This is the PETREL S scenario: laden at anchor in Lisboa, timing blocks.

```json
{
  "vessel_name": "PETREL S",
  "total_score": 57.0,
  "total_score_raw": 71.3,
  "dynamic_max_score": 125,
  "confidence": 90,
  "confidence_note": "Position confirmed via AIS. Owner ukr preference noted.",
  "recommendation": "BLOCKED",
  "short_summary": "Blocked by timing — laden at anchor Lisboa, cannot reach POC before canceling Feb 10. Owner also prefers to avoid Ukraine.",
  
  "relationship_warning": { "status": "CLEAR" },
  
  "offer_opening": {
    "intent": "blocked_info",
    "email_text": "Pls find below cargo for yr reference re mv Petrel S. We note timing is tight given her current position in Lisboa, hvr sending in case laycan extends or she frees up earlier than anticipated. Also appreciate ukr may not be first choice but cargo otherwise suits her well. Any update on her schedule wld be welcome.",
    "messenger_text": "cargo fyi for Petrel S — timing doesn't work for current dates and know ukr isn't preferred, but sending in case things change. any update on when she opens?",
    "key_selling_points": ["perfect intake fit", "PSC no issues", "company knows BS/Med"],
    "key_concerns": ["timing BLOCK — misses laycan by ~10 days", "owner prefers to avoid Ukraine"],
    "questions_to_ask": ["when does she free up?", "any change in ukr policy?"]
  },
  
  "detailed_scoring": {
    "proximity": {
      "code": "P1",
      "score": 1.2,
      "max_score": 20,
      "reasoning": "Lisboa, ~3000nm from POC. Level 12 for Small Handy (19k DWT).",
      "offer_hint": "Long ballast from Atlantic.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:proximity_scoring_matrix.txt Level 12"]
    },
    "regional_patterns": {
      "code": "P1A",
      "score": 6.9,
      "max_score": 15,
      "reasoning": "Atlantic→BS secondary pattern (base 6 × 1.15 SH multiplier).",
      "offer_hint": "Long positioning leg.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:P1A atlantic_to_black_sea"]
    },
    "owner_preferences": {
      "code": "P2",
      "score": 3.0,
      "max_score": 15,
      "reasoning": "Company comments 'ukr not preferred' — soft conflict for POC cargo.",
      "offer_hint": "Owner prefers to avoid Ukraine — check if this cargo changes view.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["companies[0].comments: 'ukr not preferred'"]
    },
    "port_accessibility": {
      "code": "P2A",
      "score": 20.0,
      "max_score": 20,
      "reasoning": "HIGH status (BV class, UK P&I, Liberia flag). Both ports Tier 3.",
      "offer_hint": null,
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:P2A Tier 3 + HIGH status"]
    },
    "cargo_preferences": {
      "code": "P3",
      "score": 0,
      "max_score": 0,
      "reasoning": "No cargo preference data — excluded.",
      "offer_hint": null,
      "decision": "excluded",
      "confidence": "n/a",
      "critical": false,
      "citations": []
    },
    "last_ports": {
      "code": "P4",
      "score": 5.5,
      "max_score": 10,
      "reasoning": "Load (POC): company has Constanta 3% (CVB, adjacent sub-region) = 5/10. Discharge (Tunisia): vessel has Mostaganem 6.6% (North Africa sub-region) = 6/10. Weighted: (5×0.6)+(6×0.4) = 5.4 → 5.5/10.",
      "offer_hint": "Company knows BS. Vessel knows North Africa.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["top_ports.company: Constanta 3%", "top_ports.vessel: Mostaganem 6.6%"]
    },
    "intake_capacity": {
      "code": "P5",
      "score": 15.0,
      "max_score": 15,
      "reasoning": "Weight 18,851t, volume 16,358t (SF 50). Final intake 16,358t fits 13,500-16,500t perfectly. Volume-limited.",
      "offer_hint": "Cargo fits perfectly.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:intake_calculator_formula.txt"]
    },
    "openarea_modifier": {
      "code": "P6",
      "score": 0,
      "max_score": 0,
      "reasoning": "open_area is null — excluded.",
      "offer_hint": null,
      "decision": "excluded",
      "confidence": "n/a",
      "critical": false,
      "citations": []
    },
    "readiness_eta": {
      "code": "P7",
      "score": 0,
      "max_score": 10,
      "reasoning": "Laden at anchor Lisboa since ~Feb 1 (ratio 8.3/8.48 = 0.978). Days since arrival: 6. Remaining ops: 1d. Ready Feb 8. Sailing Lisboa→POC: 3000nm/10.2kn = 12d. Arrives POC ~Feb 20. Layday Feb 5, canceling Feb 10. Gap +15d → BLOCK.",
      "offer_hint": "Cannot meet current laycan — send for reference only.",
      "decision": "blocked",
      "confidence": "high",
      "critical": true,
      "citations": [
        "ref:vessel_location_determination.txt: ratio 0.978 = LADEN",
        "ref:P7 CASE C: at anchor, ETA past",
        "ref:P7 gap > +7d = BLOCK"
      ]
    },
    "restrictions_compliance": {
      "code": "P8",
      "score": 20.0,
      "max_score": 20,
      "reasoning": "LOA 120m vs max 176m: PASS. Age 20yr vs max 25yr: PASS.",
      "offer_hint": null,
      "restrictions_checked": [
        "LOA 120m vs max 176m: PASS (100%)",
        "Age 20yr vs max 25yr: PASS (100%)"
      ],
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["cargo_description: loa 176, age 25"]
    }
  }
}
```

Score: active = P1(20)+P1A(15)+P2(15)+P2A(20)+P4(10)+P5(15)+P7(10)+P8(20) = 125.
Raw = 1.2+6.9+3+20+5.5+15+0+20 = 71.6. Pct = (71.6/125)×100 = 57.3%.
Recommendation = BLOCKED (P7 block overrides percentage).
Offer opening: intent = "blocked_info" — still generated!

---

## Vessel JSON — Expected Fields

```json
{
  "vessel-name": "PETREL S",
  "imo": "9363883",
  "vessel-type": "Bulk Carrier",
  "year-of-build": 2006,
  "dwt": 19101,
  "draft": 8.48,
  "current-draught": 8.3,
  "max-draught": 9.3,
  "min-draught": 4.7,
  "length-overall": "120",
  "beam": 25.0,
  "depth": 12.0,
  "grain-capacity": 817930,
  "holds": 4,
  "hatches": 4,
  "gear-details": "C-3-30",
  "classification-society": "BUREAU VERITAS",
  "flag": "Liberia",
  "pi_information": "UK P&I Club (inception 2023-02-20)",
  "destination": "Lisboa Anch., Portugal",
  "eta": "2026-02-01T07:00:00Z",
  "speed": 1.2,
  "navigation-status": "At anchor",
  "position-received": "2026-02-04T11:19:48Z",
  "vessel_current_area": "North East Atlantic Ocean",
  "average-speed": "10.2",
  "open_area": null,
  "comments": [],
  "companies": [
    { "name": "armador", "comments": ["ukr not preferred"] }
  ],
  "people": [ { "full_name": "Ebru Paylan Senkaya", "comments": [] } ],
  "top_ports": {
    "vessel": [ {"label": "Istanbul", "percent": 13.1}, ... ],
    "company": { "items": [ {"label": "Constanta", "percent": 3.0}, ... ] }
  }
}
```

### Field Usage Cheat Sheet

| Purpose | Use field | NOT field |
|---------|-----------|-----------|
| Draught ratio denominator | `draft` (8.48) | `max-draught` (9.3) |
| Current draught | `current-draught` | — |
| Vessel age | 2026 - `year-of-build` | — |
| Max draft for intake (P5) | `draft` (8.48) | `max-draught` |
| Sailing speed estimate | `average-speed` | `max-speed` |
| Port history | `top_ports.vessel` + `top_ports.company.items` | — |
| Owner preferences | `companies[].comments` | — |
| OpenArea | `open_area` | — |

---

## Key Reminders

- `total_score` = percentage (0-100), NEVER raw points
- `offer_opening` is ALWAYS generated — even for BLOCKED vessels
- BLOCKED vessels use intent = "blocked_info"
- ALL 10 criteria keys appear in detailed_scoring
- Draft → P5. Age never blocks in P8.
- Use `draft` field for draught analysis, not `max-draught`
- Check BOTH `top_ports.vessel` AND `top_ports.company.items` for P4
