# Estimation Template — JSON Output Format
## Version 4.0 — Updated 2026-02-07

This file defines the expected JSON output format for vessel scoring results.
AI should produce output matching this structure.

---

## Top-Level Structure

```json
{
  "vessel_name": "MV EXAMPLE",
  "total_score": 67.4,
  "total_score_raw": 51,
  "dynamic_max_score": 90,
  "confidence": 70,
  "confidence_note": "No data for P2, P6 — 2 criteria excluded",
  "recommendation": "CONDITIONAL",
  "short_summary": "Decent match — good positioning and timing, but vessel oversized for this parcel. Worth offering if no better options.",
  
  "relationship_warning": {
    "status": "CLEAR"
  },
  
  "offer_opening": {
    "intent": "inquire",
    "email_text": "Pls find below cargo we'd like to show for mv Example. She's well positioned at Constanta and timing looks good. Intake is a bit tight so wld appreciate yr view on qty flexibility. Kindly let us know yr interest/ideas.",
    "messenger_text": "hi, have a cargo that cld work for Example — nearby and good timing. intake bit tight but pls see below, what do you think?",
    "key_selling_points": ["proximity", "timing"],
    "key_concerns": ["intake tight due to draft restriction"],
    "questions_to_ask": ["confirm opening date"]
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
      "reasoning": "BS to Med — strong natural trade flow for coasters.",
      "offer_hint": "Natural trade pattern — mention backhaul opportunity.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["ref:P1A_REGIONAL_SCORING_RULES.txt"]
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
      "reasoning": "IACS class (BV), IG P&I. Ukraine Tier 3, Tunisia Tier 3. No PSC risk.",
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
      "reasoning": "Regular BS/Med caller. Poti 8.7%, Samsun 6.5% of calls.",
      "offer_hint": "Vessel knows the region — mention familiarity.",
      "decision": "accepted",
      "confidence": "high",
      "critical": false,
      "citations": ["top_ports data"]
    },
    
    "intake_capacity": {
      "code": "P5",
      "score": 12,
      "max_score": 15,
      "reasoning": "Weight 16,403t, volume 14,781t, draft-restricted 14,305t. Final intake 14,305t fits cargo range 13,500-16,500t.",
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

## Vessel JSON Input — Expected Fields

The vessel JSON should include these fields for scoring:

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
  "company_comments": "no ukr, spec med/bs",
  "person_comments": "",
  "vessel_comments": ""
}
```

### Draught Analysis Fields
- `current-draught`: Current draught from VesselFinder (meters)
- `draft`: Maximum summer draft from vessel specs (meters)
- `navigation-status`: AIS navigation status
- Used by: vessel_location_determination.txt, P7 scoring

---

## Relationship Warning Examples

When relationship is found:
```json
"relationship_warning": {
  "status": "WARNING",
  "type": "direct_relationship",
  "details": "Owner works directly with Bunge (from company comments)",
  "offer_guidance": "They may already have this cargo. Approach carefully."
}
```

When owner uses broker for this charterer:
```json
"relationship_warning": {
  "status": "WARNING",
  "type": "broker_relationship",
  "details": "Owner handles Cargill cargoes through Clarkson",
  "offer_guidance": "Owner may redirect to their broker. Adjust expectations."
}
```

When no relationship:
```json
"relationship_warning": {
  "status": "CLEAR"
}
```

---

## Score Calculation Example

Active criteria: P1(20), P1A(15), P2A(20), P4(10), P5(15), P7(10)
Excluded: P2(0/0), P3(0/0), P6(0/0), P8(0/0)

dynamic_max_score = 20+15+20+10+15+10 = 90
total_score_raw = 14+12+18+7+12+10 = 73
total_score = (73/90) × 100 = 81.1%
recommendation = STRONG

---

## Notes

- `total_score` is always a percentage (0-100)
- `total_score_raw` is the sum of actual points
- `dynamic_max_score` is the sum of max_score for active criteria only
- `offer_hint` can be null when not relevant for that criterion
- `restrictions_checked` array only appears in P8
- `relationship_warning` is separate from detailed_scoring
- `offer_opening` is not generated when recommendation = "BLOCKED"
