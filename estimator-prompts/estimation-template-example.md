# Estimation Template — JSON Output Format
## Version 5.0 — Updated 2026-02-07

This file defines the expected JSON output format for vessel scoring results.
AI should produce output matching this structure.

---

## Critical Rules

1. **total_score is ALWAYS a percentage** (0-100), never raw points
2. **ALL criteria must appear** in detailed_scoring, even excluded ones (0/0)
3. **Draft restrictions go to P5**, never to P8
4. **Age NEVER triggers BLOCK** in P8

---

## Top-Level Structure

```json
{
  "vessel_name": "MV EXAMPLE",
  "total_score": 81.1,
  "total_score_raw": 73,
  "dynamic_max_score": 90,
  "confidence": 85,
  "confidence_note": "High confidence. P2, P3, P6, P8 excluded (no data).",
  "recommendation": "STRONG",
  "short_summary": "Excellent match — well positioned, good timing, cargo fits nicely.",
  
  "relationship_warning": {
    "status": "CLEAR"
  },
  
  "offer_opening": {
    "intent": "attract",
    "email_text": "Pls find below cargo which suits mv Example nicely — she's well positioned and timing works. Kindly let us know yr interest/ideas.",
    "messenger_text": "have a nice cargo for Example — close by, good timing. pls see below, lmk?",
    "key_selling_points": ["proximity", "timing"],
    "key_concerns": [],
    "questions_to_ask": []
  },
  
  "detailed_scoring": {
    
    "proximity": {
      "code": "P1",
      "score": 14.0,
      "max_score": 20,
      "reasoning": "Constanta, ~150nm from Chornomorsk. Level 4 — short ballast.",
      "offer_hint": "Great positioning — lead with 'well positioned nearby'.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:proximity_scoring_matrix.txt Level 4"]
    },
    
    "regional_patterns": {
      "code": "P1A",
      "score": 12,
      "max_score": 15,
      "reasoning": "BS internal — strong natural trade flow for coasters.",
      "offer_hint": "Natural trade pattern — mention backhaul opportunity.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:P1A black_sea_internal"]
    },
    
    "owner_preferences": {
      "code": "P2",
      "score": 0,
      "max_score": 0,
      "reasoning": "No preference data available — criterion excluded.",
      "offer_hint": null,
      "decision": "excluded",
      "confidence": "n/a",
      "critical": false,
      "citations": []
    },
    
    "port_accessibility": {
      "code": "P2A",
      "score": 18,
      "max_score": 20,
      "reasoning": "IACS class (BV), IG P&I. Both ports Tier 3 — no PSC risk.",
      "offer_hint": null,
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:P2A_PSC_PORT_ACCESSIBILITY_SCORING.txt"]
    },
    
    "cargo_preferences": {
      "code": "P3",
      "score": 0,
      "max_score": 0,
      "reasoning": "No cargo preference data — criterion excluded.",
      "offer_hint": null,
      "decision": "excluded",
      "confidence": "n/a",
      "critical": false,
      "citations": []
    },
    
    "last_ports": {
      "code": "P4",
      "score": 7,
      "max_score": 10,
      "reasoning": "Regular BS/Med caller. Poti 8.7%, Samsun 6.5% — same broad region.",
      "offer_hint": "Vessel knows the region — mention familiarity.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:P4_LAST_PORTS_SCORING.txt", "top_ports data"]
    },
    
    "intake_capacity": {
      "code": "P5",
      "score": 12,
      "max_score": 15,
      "reasoning": "Weight 16,403t, volume 14,781t, draft-restricted 14,305t. Final intake 14,305t fits cargo 13,500-16,500t. Limiting factor: draft.",
      "offer_hint": "Cargo fits but at draft limit. Be transparent about qty.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:intake_calculator_formula.txt"]
    },
    
    "openarea_modifier": {
      "code": "P6",
      "score": 0,
      "max_score": 0,
      "reasoning": "No OpenArea comments — criterion excluded.",
      "offer_hint": null,
      "decision": "excluded",
      "confidence": "n/a",
      "critical": false,
      "citations": []
    },
    
    "readiness_eta": {
      "code": "P7",
      "score": 10,
      "max_score": 10,
      "reasoning": "Open Constanta Feb 12, layday Feb 14. Gap -2 days — ideal.",
      "offer_hint": "Timing perfect — highlight in opening.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:READINESSETA_TO_LOADING_PORT_SCORING.txt"]
    },
    
    "restrictions_compliance": {
      "code": "P8",
      "score": 0,
      "max_score": 0,
      "reasoning": "No restrictions set — criterion excluded.",
      "offer_hint": null,
      "restrictions_checked": [],
      "decision": "excluded",
      "confidence": "n/a",
      "critical": false,
      "citations": []
    }
  }
}
```

---

## Score Calculation Example

Active criteria: P1(20) + P1A(15) + P2A(20) + P4(10) + P5(15) + P7(10)
Excluded: P2(0/0), P3(0/0), P6(0/0), P8(0/0)

```
dynamic_max_score = 20+15+20+10+15+10 = 90
total_score_raw = 14+12+18+7+12+10 = 73
total_score = ROUND((73/90) × 100, 1) = 81.1%
recommendation = STRONG (80%+)
```

---

## Example with P8 Active (age restriction, NOT blocked)

```json
{
  "vessel_name": "MV GUANA",
  "total_score": 59.1,
  "total_score_raw": 65,
  "dynamic_max_score": 110,
  "confidence": 80,
  "confidence_note": "P2 excluded (business description only). P8 active (age restriction).",
  "recommendation": "CONDITIONAL",
  "short_summary": "Decent positioning and trade pattern. Age 2yr over limit (negotiable). Draft restriction limits intake — confirm with owner.",
  
  "relationship_warning": { "status": "CLEAR" },
  
  "offer_opening": {
    "intent": "inquire",
    "email_text": "Pls find below cargo for mv Guana. Positioning looks reasonable and she trades the area. We note age is slightly over limit but believe this is negotiable. Cld you let us know yr interest/ideas?",
    "messenger_text": "cargo for Guana — position ok, trades the area. age bit over limit but negotiable. pls see below, yr view?",
    "key_selling_points": ["regional pattern", "port familiarity"],
    "key_concerns": ["age +2yr over limit", "draft-restricted intake"],
    "questions_to_ask": ["confirm opening position"]
  },
  
  "detailed_scoring": {
    "proximity": { "code": "P1", "score": 6.0, "max_score": 20, "decision": "accepted" },
    "regional_patterns": { "code": "P1A", "score": 11.0, "max_score": 15, "decision": "accepted" },
    "owner_preferences": { "code": "P2", "score": 0, "max_score": 0, "decision": "excluded" },
    "port_accessibility": { "code": "P2A", "score": 17.0, "max_score": 20, "decision": "accepted" },
    "cargo_preferences": { "code": "P3", "score": 0, "max_score": 0, "decision": "excluded" },
    "last_ports": { "code": "P4", "score": 7.0, "max_score": 10, "decision": "accepted" },
    "intake_capacity": { "code": "P5", "score": 0, "max_score": 15, "decision": "blocked", "critical": true },
    "openarea_modifier": { "code": "P6", "score": 0, "max_score": 0, "decision": "excluded" },
    "readiness_eta": { "code": "P7", "score": 7.0, "max_score": 10, "decision": "accepted" },
    "restrictions_compliance": {
      "code": "P8",
      "score": 17.5,
      "max_score": 20,
      "reasoning": "1 restriction: Age 27 vs max 25 (+2yr, 75%). NOT blocked.",
      "restrictions_checked": ["Age 27 vs max 25: MARGINAL +2yr (75%)"],
      "decision": "accepted",
      "critical": false
    }
  }
}
```

Note: In this example, P5 is BLOCKED due to draft restriction (7.5m limit makes
loading impossible), but P8 only handles age (not draft). Age is NOT blocked.

---

## Vessel JSON Input — Expected Fields

```json
{
  "vessel-name": "ADAMAR",
  "imo": "9356517",
  "vessel-type": "General Cargo Ship",
  "dwt": "16653",
  "draft": "8.6",
  "current-draught": "8.8",
  "loa": "135.0",
  "beam": "20.4",
  "built": "2006",
  "class": "BV",
  "flag": "Turkey",
  "grain-capacity": "694700",
  "navigation-status": "Under Way Using Engine",
  "destination": "RAVENNA",
  "eta": "2026-02-13",
  "current-position": { "lat": 40.123, "lon": 17.456 },
  "vessel_area": "Adriatic Sea",
  "top_ports": [
    { "port": "Poti", "percentage": 8.7 },
    { "port": "Samsun", "percentage": 6.5 }
  ],
  "open_area": "Montoir, Feb 17",
  "company_comments": "spec med/bs",
  "person_comments": "",
  "vessel_comments": ""
}
```

### Draught Analysis Fields
- `current-draught`: Current draught from AIS (meters)
- `draft`: Maximum summer draft from vessel specs (meters)
- `navigation-status`: AIS navigation status
- Used by: vessel_location_determination.txt, P7 scoring
- Draught ratio = current-draught / draft → ballast/laden detection

---

## Relationship Warning Examples

```json
"relationship_warning": {
  "status": "WARNING",
  "type": "direct_relationship",
  "details": "Owner works directly with Bunge (from company comments)",
  "offer_guidance": "They may already have this cargo."
}
```

```json
"relationship_warning": { "status": "CLEAR" }
```

---

## Key Reminders

- `total_score` = percentage (0-100), NEVER raw points
- `total_score_raw` = sum of actual points (informational)
- `dynamic_max_score` = sum of max_score for active criteria only
- ALL 9 criteria appear in detailed_scoring (excluded show 0/0)
- Draft restrictions → P5, never P8
- Age in P8 → NEVER triggers BLOCK
- `offer_opening` not generated when BLOCKED
- `restrictions_checked` array only in P8
- `relationship_warning` is separate from detailed_scoring
