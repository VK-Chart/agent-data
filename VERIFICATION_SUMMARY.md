# CRITERIA TO VESSEL FIELDS MAPPING - FINAL
## Complete Mapping: P1-P7 Criteria → Vessel Fields → Files → Cargo Fields

**Date:** 2024-12-07  
**Status:** ✅ VERIFIED AND CORRECTED  
**Version:** 4.0 - Final corrections: removed Person structured fields, fixed P5 ranges

---

## VERIFIED INFORMATION FROM VITALY

### ✅ CONFIRMED:

**IMPORTANT NOTE ON FILE REFERENCES:**
MASTER_SCORING_SYSTEM uses legacy filenames from before reorganization. The actual current files are:
- `regional_trade_patterns.txt` (legacy) → `P1A_REGIONAL_SCORING_RULES.txt` + `REGIONAL_TRADE_PATTERNS_KB.txt` (current)
- `COMMENTS_PROCESSING_GUIDE.txt` (legacy) → `COMMENTS_PROCESSING_RULES.txt` + `PROMPTS.txt` + `EXAMPLES.txt` (current)
- `cargo_export_map.txt` (planned but not created - OK)
- `learning_session_1.txt` (planned but not created - OK)

This mapping uses **current actual filenames** throughout. Legacy files are preserved in `/Legacy/` folder for reference.

---

1. **`draft` field** - Already populated for vessels in production
   - Example document shown had `draft: 0` but this is NOT representative
   - Real vessels have populated summer draft values (5-10m typically)
   - Source: MarineMAN "Draft" field = `T_summer`

2. **`port_calls` data** - Already exists for companies
   - Company-level statistics: Top ports/countries/regions by visits
   - Visual example: See Vitaly's screenshot showing port distribution charts
   - Will be added to vessel-level API response by developer

3. **Structured preference fields** - UI settings for P2 and P3
   - `vessel.regional_preferences` - Vessel-level regional settings (HIGHEST priority)
   - `companies[].regional_preferences` - Company-level regional settings
   - `vessel.cargo_preferences` - Vessel-level cargo type settings (HIGHEST priority)
   - `companies[].cargo_preferences` - Company-level cargo type settings
   - `special_requirements` - Additional cargo requirements
   - These are STRUCTURED fields separate from parsed comments
   - **Priority:** Vessel structured > Company structured > Vessel parsed > Company parsed
   - **Note:** Person preferences exist only in `people[].comments` (parsed), not as structured fields

4. **P6 relationship with P1A**
   - P6 (OpenArea Comments) is closely related to P1A (Regional Patterns)
   - Current voyage strategy (P6) builds on regional market logic (P1A)
   - See note in P6 section below

---

## P1: PROXIMITY (Geographic Distance) [0-20 points]

### VESSEL FIELDS REQUIRED:

#### Current Position:
```javascript
vessel_current_area: 'Black Sea'        // Current zone/region
open_area: {
  name: 'tbsea',                        // Can be: port, region, country, sea, global area
  start_at: ISODate('2025-08-23'),      // Open date from
  end_at: ISODate('2025-08-23'),        // Open date to (can be same = exact date)
  comments: []                           // OpenArea comments (used in P6)
}
destination: 'Eregli (Erdemir), Turkey' // Where heading
```

#### Vessel Size (for distance penalties):
```javascript
dwt: 3262                               // For size classification
```

#### Geopolitical:
```javascript
flag: 'Antigua & Barbuda'               // For sanctions/restrictions
```

### CARGO FIELDS REQUIRED:
```javascript
loading_port: 'Odessa'                  // Port name
loading_region: 'Black Sea'             // Region fallback
loading_country: 'Ukraine'              // Country fallback
```

### FILES USED:
**PRIMARY:**
- `proximity_scoring_matrix.txt` - Main algorithm, internal [0-50] → P1 [0-20]

**AUXILIARY:**
- `vessel_location_determination.txt` - Determine vessel position (open > destination > current)
- `vessel_size_classification.txt` - DWT → Size category
- `CERDIseadistance.xlsx` - Sea distance calculations
- `port_restrictions_map.txt` - Port blocking (size > capacity = IMPOSSIBLE)

**CROSS-CUTTING:**
- `MASTER_SCORING_SYSTEM.txt` - P1 definition

### SCORING LOGIC:
1. Determine vessel location (priority: open_area > destination > current_area)
2. Get vessel size category from DWT
3. Calculate distance to loading port
4. Find proximity level (1-18) from matrix
5. Apply distance penalty per 100nm (size-dependent)
6. Check geopolitical blocks (Ukraine ↔ Russia = IMPOSSIBLE)
7. Scale internal score [0-50] → P1 [0-20]

---

## P1A: REGIONAL PATTERNS (Market Logic) [0-15 points]

### VESSEL FIELDS REQUIRED:

#### Position:
```javascript
vessel_current_area: 'Black Sea'        // Starting region
open_area.name: 'tbsea'                 // Opening region
dwt: 3262                               // For vessel size rules
'vessel-type': 'General Cargo Ship'     // Vessel type
```

### CARGO FIELDS REQUIRED:
```javascript
loading_region: 'Black Sea'             // Loading region
loading_country: 'Ukraine'              // Loading country
discharge_region: 'East Med'            // Discharge region
discharge_country: 'Egypt'              // Discharge country
commodity_type: 'Wheat'                 // Cargo type (affects patterns)
month: 9                                // For seasonal adjustments
```

### FILES USED:
**PRIMARY:**
- `P1A_REGIONAL_SCORING_RULES.txt` - Scoring algorithm
- `REGIONAL_TRADE_PATTERNS_KB.txt` - Knowledge base (RAG source)

**AUXILIARY:**
- `vessel_size_classification.txt` - Size categories for patterns
- `vessel_location_determination.txt` - Determine region

**CROSS-CUTTING:**
- `MASTER_SCORING_SYSTEM.txt` - P1A definition

### SCORING LOGIC:
1. Determine vessel category (SC/C/SH/H/SS/SM/PM)
2. Identify region pair (vessel_region → cargo_region)
3. Find pattern in REGIONAL_TRADE_PATTERNS_KB
4. Apply vessel_size_rules from pattern
5. Add seasonal adjustments (grain season, winter)
6. Add port-specific modifiers
7. Return P1A [0-15]

**NOTE:** P1A provides market context for P6 (OpenArea comments). When vessel owner states preferences in P6, they often align with or deviate from typical P1A patterns for strategic reasons.

---

## P2: REGIONAL PREFERENCES (Manual Owner Settings) [0-15 points]

### VESSEL FIELDS REQUIRED:

#### Structured Preference Fields (UI settings):
```javascript
// Vessel-level preferences (HIGHEST priority)
vessel.regional_preferences: {          // ⭐ STRUCTURED UI settings
  specializes: ['Turkey', 'Greece'],    // +15 points regions
  prefers: ['Italy', 'Spain'],          // +10 points regions
  can_work: ['Romania', 'Bulgaria'],    // +5 points regions
  avoids: ['Egypt'],                    // -10 points regions
  cannot_work: ['Libya']                // -15 points regions
}

// Company-level preferences
companies[].regional_preferences: {}    // Company-wide settings
```

#### Comments Sources (parsed for additional preferences):
```javascript
companies[].comments: []                // "Italy ok", "no Egypt"
comments: ['no ukr']                    // Vessel-specific notes
people[].comments: []                   // Person preferences
```

### CARGO FIELDS REQUIRED:
```javascript
loading_region: 'Black Sea'             // Match against preferences
loading_country: 'Ukraine'              // Match against preferences
discharge_region: 'East Med'            // Match against preferences
discharge_country: 'Egypt'              // Match against preferences
```

### FILES USED:
**PRIMARY:**
- `COMMENTS_PROCESSING_RULES.txt` - Business logic for parsing
- `COMMENTS_PROCESSING_PROMPTS.txt` - AI instructions
- `COMMENTS_PROCESSING_EXAMPLES.txt` - Training examples

**CROSS-CUTTING:**
- `MASTER_SCORING_SYSTEM.txt` - P2 definition
- `SCORING_RECALCULATION_TRIGGERS.yaml` - UI fields definitions

### COMMENT PATTERNS TO DETECT:
```
Positive: "Italy ok", "Greece excellent", "EU ok", "Black Sea preferred"
Negative: "no Egypt", "avoids Ukraine", "no Continent"
Specialization: "spec: Turkish domestic cargoes"
```

### PREFERENCE LEVELS (Structured Fields):
```
specializes: +15 points     // Company/vessel specializes in region
prefers: +10 points         // Prefers this region
can_work: +5 points         // Acceptable region
neutral: 0 points           // No preference set
avoids: -10 points          // Tries to avoid
cannot_work: -15 points     // Cannot/will not work
```

### PRIORITY HIERARCHY (Conflict Resolution):
```
1. Vessel structured settings (vessel.regional_preferences)
2. Company structured settings (companies[].regional_preferences)
3. Vessel parsed comments (vessel.comments)
4. Company parsed comments (companies[].comments)
```

**Note:** Person preferences exist only in `people[].comments` (parsed), not as structured fields. They can be parsed but have lower priority than company comments.

### SCORING LOGIC:
1. Check vessel structured fields first (vessel.regional_preferences)
2. If no vessel structured preference, check company structured fields
3. Apply preference level for cargo regions from structured fields
4. If no structured preference, parse vessel comments
5. If no vessel comment preference, parse company comments
6. Identify: specializes/prefers/can_work/avoids/cannot_work regions
7. Match cargo regions against all preferences
8. Apply priority hierarchy (vessel structured > company structured > vessel parsed > company parsed)
9. Return P2 [0-15]

---

## P3: CARGO PREFERENCES (Cargo Type Compatibility) [0-15 points]

### VESSEL FIELDS REQUIRED:

#### Structured Preference Fields (UI settings):
```javascript
// Vessel-level preferences (HIGHEST priority)
vessel.cargo_preferences: {             // ⭐ STRUCTURED UI settings
  specializes_in: ['Grain', 'Wheat'],   // +15 points cargo types
  regularly_carries: ['Fertilizers'],   // +10 points cargo types
  can_carry: ['Steel', 'Scrap'],        // +5 points cargo types
  avoids: ['Coal'],                     // -10 points cargo types
  cannot_carry: ['IMO', 'Dangerous']    // BLOCK cargo types
}

// Company-level preferences
companies[].cargo_preferences: {}       // Company-wide settings
companies[].special_requirements: {}    // Additional cargo requirements
```

#### Comments Sources (parsed for additional preferences):
```javascript
companies[].comments: []                // Cargo specialization
comments: ['no ukr']                    // Cargo capabilities
people[].comments: []                   // Cargo preferences
```

#### Technical Capabilities:
```javascript
'vessel-type': 'General Cargo Ship'     // Type
'gear-type': 0                          // 0 = gearless
'gear-construction-details': 'Gearless' // Gear info
'imo-fitted': null                      // IMO capability
```

### CARGO FIELDS REQUIRED:
```javascript
commodity_type: 'Wheat'                 // Cargo type
commodity_group: 'Grain'                // General category
stowage_factor: 0.68                    // SF value
cargo_form: 'bulk'                      // bulk/bagged/etc.
```

### FILES USED:
**PRIMARY:**
- `COMMENTS_PROCESSING_RULES.txt` - Cargo preference parsing
- `COMMENTS_PROCESSING_PROMPTS.txt` - AI instructions
- `COMMENTS_PROCESSING_EXAMPLES.txt` - Training examples

**AUXILIARY:**
- `port_restrictions_map.txt` - Gear requirements (sugar in bags → gear required)

**CROSS-CUTTING:**
- `MASTER_SCORING_SYSTEM.txt` - P3 definition
- `vessel_size_classification.txt` - Size-specific cargo compatibility
- `SCORING_RECALCULATION_TRIGGERS.yaml` - UI fields definitions

### COMMENT PATTERNS TO DETECT:
```
Negative: "no grains", "no steel", "avoid fertilizers"
Positive: "prefer high SF cargoes", "only grains", "spec: grain trades"
Technical: "IMO-fitted", "gear available", "cement outfit"
```

### PREFERENCE LEVELS (Structured Fields):
```
specializes_in_cargo: +15 points    // Specializes in this cargo type
regularly_carries: +10 points       // Regularly carries
can_carry: +5 points                // Technically capable
neutral: 0 points                   // No preference
avoids_cargo: -10 points            // Tries to avoid
cannot_carry_technically: BLOCK     // Technical impossibility
```

### PRIORITY HIERARCHY (Conflict Resolution):
```
1. Vessel structured settings (vessel.cargo_preferences)
2. Company structured settings (companies[].cargo_preferences)
3. Vessel parsed comments (vessel.comments)
4. Company parsed comments (companies[].comments)
5. Technical capabilities (gear, IMO, vessel-type)
```

**Note:** Person preferences exist only in `people[].comments` (parsed), not as structured fields. Technical capabilities (gear, IMO) are checked last as hard constraints.

### SCORING LOGIC:
1. Check vessel structured fields first (vessel.cargo_preferences)
2. If no vessel structured preference, check company structured fields
3. Apply preference level for cargo type from structured fields
4. If no structured preference, parse vessel comments
5. If no vessel comment preference, parse company comments
6. Check technical compatibility (IMO, gear, etc.)
7. Match cargo type against all preferences
8. Check port gear requirements (from port_restrictions_map)
9. Apply priority hierarchy (vessel structured > company structured > vessel parsed > company parsed > technical)
10. Return P3 [0-15] or BLOCK if cannot_carry_technically

---

## P4: LAST PORTS (Port Familiarity) [0-10 points]

### VESSEL FIELDS REQUIRED:

#### Port History (SEPARATE TABLE/API):
```javascript
// ⭐ COMPANY-LEVEL (already exists - see Vitaly's screenshot)
companies[].port_calls: {
  ports: [                              // Top ports by visits
    { name: 'Istanbul', visits: 75, percentage: 8.2 },
    { name: 'Kartal', visits: 53, percentage: 5.8 },
    { name: 'Iskenderun', visits: 36, percentage: 3.9 }
  ],
  countries: [                          // Top countries by visits
    { name: 'Turkey', visits: 296, percentage: 32.4 },
    { name: 'Egypt', visits: 80, percentage: 8.8 },
    { name: 'Israel', visits: 62, percentage: 6.8 }
  ],
  regions: [                            // Top regions by visits
    { name: 'European Union', visits: 267, percentage: 29.2 },
    { name: 'Emed', visits: 228, percentage: 25 },
    { name: 'aegean-marmara range', visits: 222, percentage: 24.3 }
  ]
}

// ⭐ VESSEL-LEVEL (will be added by developer)
port_calls: {
  history: [
    { port: 'Odessa', last_visit: '2024-10-15', total_visits: 5 },
    { port: 'Constanta', last_visit: '2024-11-20', total_visits: 3 }
  ],
  statistics: {
    regions: { 'Black Sea': 15, 'Med': 8 },
    countries: { 'Ukraine': 5, 'Romania': 3, 'Turkey': 7 }
  }
}

// Current vessel document field (not populated):
'recent-ports-call': null               // ⚠️ Will be replaced by port_calls above
```

**VISUAL REFERENCE:**  
See Vitaly's screenshot showing company fleet statistics with pie charts for:
- Top 10 Ports (Istanbul 8.2%, Kartal 5.8%, Iskenderun 3.9%, etc.)
- Top Countries (Turkey 32.4%, Egypt 8.8%, Israel 6.8%, etc.)
- Top Regions (European Union 29.2%, Emed 25%, aegean-marmara 24.3%, etc.)

### CARGO FIELDS REQUIRED:
```javascript
loading_port: 'Odessa'                  // Check familiarity
loading_region: 'Black Sea'             // Region familiarity
discharge_port: 'Alexandria'            // Discharge familiarity
```

### FILES USED:
**PRIMARY:**
- `MASTER_SCORING_SYSTEM.txt` - P4 definition section

**AUXILIARY:**
- `vessel_size_classification.txt` - Size-specific return patterns

### SCORING LOGIC:
1. Query port_calls data for vessel history
2. Calculate port visit frequency (loading + discharge ports)
3. Calculate region familiarity (loading + discharge regions)
4. Recent visits boost score
5. Never visited = slight penalty, frequent visits = bonus
6. Return P4 [0-10]

### ⚠️ IMPLEMENTATION NOTE:
**Developer action required:**
- Add vessel-level port_calls to API response
- Format similar to company-level structure shown in screenshot
- Include visit counts, last visit dates, regional statistics

---

## P5: INTAKE CAPACITY (Weight/Volume Match) [0-15 points]

### VESSEL FIELDS REQUIRED:

#### Critical Fields (for calculation):
```javascript
// ⭐ Summer draft field
draft: 6.2                              // T_summer (summer design draft in meters)
                                        // ✅ Populated in production vessels
                                        // Source: MarineMAN "Draft" field
                                        // Typical values: 5-10m

dwt: 3262                               // Deadweight tonnage
beam: 14                                // Beam (width) in meters
'length-overall': '90'                  // LOA in meters
'length-bp': '85'                       // Lpp in meters (preferred over LOA)
'grain-capacity': 171984                // Grain space in m³
'vessel-type': 'General Cargo Ship'     // For CWP coefficient (bulk vs other)
```

#### Manual Override (from comments):
```javascript
comments: [
  'Real intake: 5800t constanta-egypt', // Manual intake override
  'corn Izmail 4800t draft 7m'          // Port-specific intake
]
```

### CARGO FIELDS REQUIRED:
```javascript
quantity: 5000                          // Required cargo tonnage
stowage_factor: 0.68                    // SF in ft³/ton (for volume limit)
loading_port: 'Constanta'               // For port restrictions
discharge_port: 'Alexandria'            // For discharge restrictions
```

### PORT DATA REQUIRED (from port_restrictions_map.txt):
```javascript
port_draft_limit: 9.5                   // Max draft in meters
water_type: 'fresh'                     // fresh/brackish/sea
seasonal_restrictions: []               // Ice, low water, etc.
```

### FILES USED:
**PRIMARY:**
- `intake_calculator_formula.txt` - Complete calculation algorithm

**AUXILIARY:**
- `port_restrictions_map.txt` - Draft limits, water type, seasonal rules
- `vessel_size_classification.txt` - Bunker estimates, speed
- `COMMENTS_PROCESSING_RULES.txt` - Manual intake override parsing
- `COMMENTS_PROCESSING_PROMPTS.txt` - AI instructions for parsing
- `COMMENTS_PROCESSING_EXAMPLES.txt` - Intake parsing examples

**CROSS-CUTTING:**
- `MASTER_SCORING_SYSTEM.txt` - P5 definition

### INTAKE CALCULATION STEPS:

**STEP 1: Validate Critical Data**
```
Required: draft (T_summer), beam, port_draft_limit, water_type, LOA or Lpp
If missing → ERROR, cannot calculate
```

**STEP 2: Calculate Hydrostatic Parameters**
```
Lpp_used = Lpp OR (0.965 × LOA for bulk) OR (0.97 × LOA for other)
CWP = 0.87 (bulk/DWT>20000) OR 0.80 (other)
TPC_salt = (Lpp × Beam × CWP) / 100
TPC_rho = TPC_salt × (water_density / 1.025)
Delta_summer = 1.025 × Lpp × Beam × T_summer × 0.74
```

**STEP 3: Calculate Allowable Draft**
```
UKC = 0.30m (under keel clearance)
SQUAT = 0.10m (squat allowance)
T_allow = port_draft_limit - UKC - SQUAT
T_allow_SW = T_allow adjusted for water density
```

**STEP 4: Calculate Draft Reduction**
```
dT = MAX(0, T_summer - T_allow_SW)
Loss_t = dT × TPC_rho × 100
```

**STEP 5: Calculate Weight-Limited Intake**
```
DWT_available = DWT - Loss_t
ROB = DWT × 0.04 (4% for bunkers/stores)
Intake_weight = DWT_available - ROB
```

**STEP 6: Calculate Volume-Limited Intake (if applicable)**
```
IF grain_m³ AND stowage_factor exist:
  SF_m3_per_t = stowage_factor × 0.02832  // Convert ft³/t to m³/t
  Intake_volume = grain_m³ / SF_m3_per_t
  Final_intake = MIN(Intake_weight, Intake_volume)
ELSE:
  Final_intake = Intake_weight
```

**STEP 7: Check Manual Override**
```
IF vessel.comments contains "Real intake: XXXXt":
  Extract intake value from comments
  OVERRIDE calculated intake with manual value
  Note: Manual intake has HIGHEST priority
```

**STEP 8: Calculate P5 Score**
```
Ratio = Final_intake / cargo_quantity

IF ratio >= 1.10: P5 = 15 (comfortable margin ≥110%)
IF ratio 1.00-1.10: P5 = 12 (exact fit 100-110%)
IF ratio 0.95-1.00: P5 = 8 (tight fit 95-100%)
IF ratio 0.90-0.95: P5 = 3 (requires negotiation 90-95%)
IF ratio < 0.90: BLOCK (insufficient capacity <90%)
```

---

## P6: OPEN AREA COMMENTS (Current Voyage Preferences) [-50 to +25 points]

### VESSEL FIELDS REQUIRED:

#### OpenArea Comments:
```javascript
open_area: {
  name: 'tbsea',
  start_at: ISODate('2025-08-23'),
  end_at: ISODate('2025-08-23'),
  comments: [                           // PRIMARY SOURCE for P6
    'no grains',
    'prefer nearby only',
    'seeking short trip'
  ]
}

companies[].comments: []                // Company OpenArea notes (if exists)
people[].comments: []                   // Person voyage notes
```

### CARGO FIELDS REQUIRED:
```javascript
commodity_type: 'Wheat'                 // Match cargo-type signals
loading_region: 'Black Sea'             // Match geography signals
discharge_region: 'East Med'            // Match route signals
stowage_factor: 0.68                    // Match SF preferences
voyage_duration_days: 5                 // Match trip length signals
distance_ballast: 800                   // Match ballast distance signals
```

### FILES USED:
**PRIMARY:**
- `OPEN_AREA_COMMENTS_SCORING.txt` - Main P6 algorithm

**AUXILIARY:**
- `COMMENTS_PROCESSING_RULES.txt` - OpenArea parsing logic
- `COMMENTS_PROCESSING_PROMPTS.txt` - AI instructions
- `COMMENTS_PROCESSING_EXAMPLES.txt` - Training examples

**CROSS-CUTTING:**
- `MASTER_SCORING_SYSTEM.txt` - P6 definition
- `P1A_REGIONAL_SCORING_RULES.txt` - Regional pattern context
- `REGIONAL_TRADE_PATTERNS_KB.txt` - Market knowledge base

### RELATIONSHIP WITH P1A:
**P6 builds on P1A regional market logic:**
- P1A: "What is the typical market pattern for this vessel size and route?"
- P6: "What is the owner's CURRENT strategy vs typical patterns?"

**Examples:**
- P1A might show: "Libya → Black Sea grain is typical for coasters"
- P6 owner says: "no Black Sea this voyage" (deviating from typical pattern)
- Result: P1A stays positive (pattern is valid), P6 penalizes (owner preference conflict)

**Integration:**
- Use P1A knowledge base to understand WHY certain comments appear
- "seeking backhaul to X" → Check P1A for typical backhaul routes
- "repositioning" → Understand against P1A regional patterns

### SIGNAL CATEGORIES:

**1. Cargo-Type Signals:**
```
Negative: "no grains", "no steel", "avoid fertilizers"
Positive: "only grains", "prefer high SF", "fertilizers preferred"
Technical: "high SF only", "no low SF"
```

**2. Geography Signals:**
```
Negative: "no Black Sea", "avoid Egypt", "no long ballast"
Positive: "only Black Sea", "East Med preferred", "nearby only"
Backhaul: "seeking backhaul to X", "returning to Y"
```

**3. Strategy Signals:**
```
Trip length: "short trip only", "long voyage ok"
Positioning: "repositioning to X", "moving to Y area"
Market: "waiting for rates", "seeking firm cargo"
```

### SCORING LOGIC:

**Full Conflict (-40 to -50):**
```
"no grains" + cargo is grain → -45
"no Black Sea" + loading is Black Sea → -45
Multiple conflicts stack (clamped at -50)
```

**Soft Conflict (-10 to -25):**
```
"prefer close cargo" + 800nm ballast → -15
"short trip only" + 10-day voyage → -10
```

**Neutral (0):**
```
No relevant comments → 0
```

**Positive Match (+10 to +15):**
```
"prefer high SF" + SF=1.45 → +12
"East Med ok" + cargo to Egypt → +10
```

**Perfect Match (+20 to +25):**
```
"looking for short trip to Egypt with high SF" + exact match → +25
```

**BLOCKING RULE:**
```
IF P6 ≤ -40: Do NOT send offer
Note: This is a soft block, cargo still shown with reasoning
```

---

## P7: READINESS/ETA (Timing Match) [0-10 points]

### VESSEL FIELDS REQUIRED:

#### Position & Timing:
```javascript
open_area: {
  name: 'tbsea',                        // Open position
  start_at: ISODate('2025-08-23'),      // Open date from
  end_at: ISODate('2025-08-23')         // Open date to
}
destination: 'Eregli, Turkey'           // Current destination
eta: ISODate('2025-08-26T15:00')        // ETA to destination
vessel_current_area: 'Black Sea'        // Current area (fallback)
```

#### Laden/Ballast Detection:
```javascript
draft: 6.2                              // T_summer (design draft)
                                        // ✅ Populated in production
'current-draught': 3.8                  // Current draft
'navigation-status': 'Under way'        // Status hint
```

#### Size (for service speed):
```javascript
dwt: 3262                               // For size category → speed
```

### CARGO FIELDS REQUIRED:
```javascript
loading_port: 'Odessa'                  // Destination for ETA
late_date: ISODate('2024-08-28')        // Laycan start
cancelling_date: ISODate('2024-09-02')  // Laycan end
```

### FILES USED:
**PRIMARY:**
- `READINESSETA_TO_LOADING_PORT_SCORING.txt` - Main P7 algorithm

**AUXILIARY:**
- `vessel_location_determination.txt` - Starting point determination
- `vessel_size_classification.txt` - Service speed by size
- `CERDIseadistance.xlsx` - Sea distance calculation

**CROSS-CUTTING:**
- `MASTER_SCORING_SYSTEM.txt` - P7 definition

### ETA CALCULATION STEPS:

**STEP 1: Determine Starting Point**
```
Priority 1: open_area.name + open_area.start_at
Priority 2: destination + eta
Priority 3: vessel_current_area + current time
```

**STEP 2: Determine Laden vs Ballast**
```
Laden if: current_draft >= 0.85 × draft (T_summer)
Ballast if: current_draft <= 0.60 × draft (T_summer)
Uncertain: 0.60-0.85 range → use patterns

✅ Production vessels have draft populated
```

**STEP 3: Calculate Start Time**
```
IF ballast:
  start_time = open_area.start_at
ELSE IF laden:
  discharge_time = cargo_quantity / loading_rate × SHEX_coefficient
  start_time = eta + discharge_time
```

**STEP 4: Select Service Speed**
```
Small Coasters (0-4000 DWT): 8 knots
Coasters (4001-17000 DWT): 10 knots
Larger vessels (17001+ DWT): 12 knots
```

**STEP 5: Calculate Distance**
```
Distance = sea_distance(start_point, loading_port)
Source: CERDIseadistance.xlsx
```

**STEP 6: Calculate ETA to Loading Port**
```
Days = distance_nm / (speed_kn × 24)
ETA_to_load = start_time + days
```

**STEP 7: Compare vs Laycan**
```
Perfect fit (9-10 points):
  late_date <= ETA_to_load <= cancelling_date

Slightly early (7-8 points):
  ETA_to_load 1-3 days before late_date

Slightly late (4-6 points):
  ETA_to_load 1-3 days after cancelling_date

Too early or late (1-3 points):
  >3 days difference

Unknown (0 points):
  Cannot calculate ETA
```

---

## CROSS-CUTTING FILES SUMMARY

### 1. MASTER_SCORING_SYSTEM.txt
**Affects:** ALL P1-P7
- Single source of truth
- Defines all criteria ranges
- Laravel ↔ LangChain contract

### 2. vessel_size_classification.txt
**Affects:** P1, P1A, P3, P4, P5, P7
- DWT → Size category (SC/C/SH/H/SS/SM/PM)
- Size-specific behavior

### 3. vessel_location_determination.txt
**Affects:** P1, P7, (P1A indirect)
- Vessel position logic
- Priority: open > destination > current

### 4. COMMENTS_PROCESSING_*.txt (3 files)
**Affects:** P2, P3, P5, P6
- Rules, Prompts, Examples
- Comment parsing logic

### 5. port_restrictions_map.txt
**Affects:** P1, P3, P5
- Port blocking, gear requirements, draft limits

### 6. CERDIseadistance.xlsx
**Affects:** P1, P7
- Sea distance calculations

### 7. SCORING_RECALCULATION_TRIGGERS.yaml
**Affects:** ALL P1-P7
- Defines WHEN to recalculate scores
- Field change triggers
- UI fields definitions (regional_preferences, cargo_preferences, special_requirements)

---

## MESSAGE GENERATION FILES (NOT SCORING)

### 1. ABBREVIATIONS_KNOWLEDGE_BASE.txt
**Purpose:** Abbreviations for emails/messages
**Usage:** Offer text generation

### 2. emailsexamples.txt
**Purpose:** Email/WhatsApp templates
**Usage:** Message formatting

---

## CRITICAL ACTION ITEMS FOR DEVELOPER

### Priority 1: HIGH (Enable P4)
- [ ] **Add `port_calls` to vessel API response**
  - Use company-level structure as reference (see screenshot)
  - Include vessel-specific history: visits, dates, statistics
  - Format: See P4 section above

### Priority 2: MEDIUM (Verify data quality)
- [ ] **Verify `draft` field populated for all vessels**
  - Spot check: Ensure no vessels have draft: 0 in production
  - Source: MarineMAN "Draft" field
  - If any missing → populate from MarineMAN

### Priority 3: MEDIUM (Field mapping)
- [ ] **Create field name mapping layer:**
  ```
  File expects → Vessel document has
  T_summer → draft
  current_draft → current-draught
  eta_destination → eta
  open_date_from → open_area.start_at
  open_date_to → open_area.end_at
  ```

### Priority 4: LOW (UI fields)
- [ ] **Verify structured preference fields exist:**
  - `regional_preferences` (company/vessel/person)
  - `cargo_preferences` (company/vessel/person)
  - `special_requirements` (company)

### Priority 5: LOW (Documentation)
- [ ] Update API docs with field mappings
- [ ] Document preference field structure

---

## SUMMARY TABLE: ALL VESSEL FIELDS BY CRITERION

| Vessel Field | P1 | P1A | P2 | P3 | P4 | P5 | P6 | P7 | Notes |
|--------------|----|----|----|----|----|----|----|----|-------|
| `dwt` | ✓ | ✓ | | ✓ | | ✓ | | ✓ | Size classification |
| `draft` (T_summer) | | | | | | **✓** | | **✓** | ✅ Populated in production |
| `vessel_current_area` | ✓ | ✓ | | | | | | ✓ | Current zone |
| `open_area.name` | ✓ | ✓ | | | | | ✓ | ✓ | Port/region/country/sea |
| `open_area.start_at` | | | | | | | | ✓ | Open date from |
| `open_area.end_at` | | | | | | | | ✓ | Open date to |
| `open_area.comments` | | | | | | | **✓** | | P6 primary source |
| `destination` | ✓ | | | | | | | ✓ | Where heading |
| `eta` | | | | | | | | ✓ | ETA to destination |
| `current-draught` | | | | | | | | ✓ | Current draft |
| `navigation-status` | | | | | | | | ✓ | Under way/etc |
| `vessel-type` | | ✓ | | ✓ | | ✓ | | | Type |
| `length-overall` | | | | | | ✓ | | | LOA |
| `length-bp` | | | | | | ✓ | | | Lpp (preferred) |
| `beam` | | | | | | ✓ | | | Width |
| `grain-capacity` | | | | | | ✓ | | | Volume |
| `gear-type` | | | | ✓ | | | | | Gearless = 0 |
| `gear-construction-details` | | | | ✓ | | | | | Gear info |
| `flag` | ✓ | | | | | | | | Sanctions |
| `vessel.regional_preferences` | | | **✓** | | | | | | ⭐ Vessel structured |
| `companies[].regional_preferences` | | | **✓** | | | | | | ⭐ Company structured |
| `vessel.cargo_preferences` | | | | **✓** | | | | | ⭐ Vessel structured |
| `companies[].cargo_preferences` | | | | **✓** | | | | | ⭐ Company structured |
| `special_requirements` | | | | ✓ | | | | | ⭐ Company structured |
| `comments` | | | ✓ | ✓ | | ✓ | ✓ | | Vessel parsed |
| `companies[].comments` | | | ✓ | ✓ | | | ✓ | | Company parsed |
| `people[].comments` | | | ✓ | ✓ | | | ✓ | | Person parsed |
| `port_calls` | | | | | **✓** | | | | ⚠️ Developer to add |

**Legend:**
- ✓ = Used by criterion
- **✓** = CRITICAL for criterion or NEW structured field
- ⭐ = Structured UI field (not comments)
- ⚠️ = Action required

---

**STATUS:** ✅ COMPLETE - Ready for implementation
**UPDATED:** Added P2/P3 structured preference fields, corrected draft status, added port_calls visual reference
