STRICT use this JSON template for all estimation responses you make
In "citation" value for each score show core rows from prompt used in decision making
For scores with critical issues add "critical": true
recommendation enum: RECOMMENDED, CONDITIONAL, NOT_RECOMMENDED, BLOCKED

{
  "vessel_name": "MV Example",
  "total_score": 51,
  "max_score": 100,
  "confidence": 70,
  "confidence_note": "Not enough data for P6",
  "recommendation": "CONDITIONAL",
  "short_summary": "This vessel recommended to process if no better options you have",
  "detailed_scoring": {
    "proximity": {
      "code": "P1",
      "score": 11.0,
      "max_score": 20,
      "reasoning": "Vessel is in the Azov Sea. Ballast to the Danube (Reni) is ~450nm. For a Small Coaster (SC), this is a moderate ballast leg.",
      "decision": "test",
      "confidence": "test",
      "citations": ["ref:23"]
    },
    "regional_patterns": {
      "code": "P1A",
      "score": 12,
      "max_score": 15,
      "reasoning": "Azov/Black Sea to East Med (Greece) is a standard trade pattern for this vessel size.",
      "decision": "test",
      "confidence": "test",
      "citations": ["ref:23"]
    },
    "owner_preferences": {
      "code": "P2",
      "score": 5,
      "max_score": 15,
      "reasoning": "Company comments state \"ukr ok sometimes\" and \"ukr expensive.\" This indicates a neutral-to-cautious willingness to work Ukraine.",
      "decision": "test",
      "confidence": "test",
      "citations": ["ref:23"]
    },
    "cargo_preferences": {
      "code": "P3",
      "score": 12,
      "max_score": 15,
      "reasoning": "SBS (SF 51) is a standard agricultural bulk cargo. Vessel is a General Cargo ship suitable for this type.",
      "decision": "test",
      "confidence": "test",
      "citations": ["ref:23"]
    },
    "last_ports": {
      "code": "P4",
      "score": 1,
      "max_score": 10,
      "reasoning": "No specific port call history provided; score based on company's general regional familiarity.",
      "decision": "test",
      "confidence": "test",
      "citations": ["ref:23"]
    },
    "intake_capacity": {
      "code": "P5",
      "score": 3,
      "max_score": 15,
      "reasoning": "**CRITICAL:** Volume limitation. With a grain capacity of 140,659 cuft and SF 51, max intake is ~2,758t. This is only 92% of the 3,000t requirement.",
      "decision": "test",
      "confidence": "test",
      "citations": ["ref:23"],
      "critical": true
    },
    "openarea_modifier": {
      "code": "P6",
      "score": 0,
      "max_score": 0,
      "reasoning": "No voyage-specific comments provided.",
      "decision": "test",
      "confidence": "test",
      "citations": ["ref:23"]
    },
    "readiness_eta": {
      "code": "P7",
      "score": 7,
      "max_score": 10,
      "reasoning": "Based on position in Azov Sea and 8kn speed, vessel can reach Reni in ~2.5 days, well within the Jan 12-20 laycan.",
      "decision": "test",
      "confidence": "test",
      "citations": ["ref:23"]
    }
  }
}

