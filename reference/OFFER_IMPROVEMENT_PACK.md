# OFFER IMPROVEMENT PACK v1.0
# Reviewed and approved by Vitaly — 2026-04-09
# Date: 2026-04-09

---

## PART A — ABBREVIATED VOCABULARY FOR OFFERS (filtered from full KB)

Only abbreviations that appear IN the offer text itself (email/messenger).
CP terms, technical terms, port codes — excluded (not needed in offer body).

```
MANDATORY — always use, never write full word:
  about        → abt
  your         → yr
  have         → hv
  vessel       → vsl
  following    → foll
  subject to   → sub
  discharge    → disch
  loading      → ldg
  completing   → compl
  available    → avail / free
  schedule     → sched
  interested   → intd
  ballast      → blst
  however      → hvr
  please       → pls
  regards      → rgds
  thanks       → tks
  cargo        → cgo
  charterer    → chrtr
  freight      → frt
  deadweight   → dwt
  stowage factor → sf
  without guarantee → wog
  total        → ttl
  that         → tt
  good         → gd
  metric tonne → mt
  owner        → onw / ownr
  breakbulk    → bb
  before       → b4
  information  → (DELETE — never use "for your information", say "fyi")

COMMON IN OFFERS — use when natural:
  opening      → opens
  position     → posn
  approximately → abt
  inclusive    → incl
  exclusive   → excl
  commission  → (use number only: "2.5" or "2.5 add")
  cubic       → cubic (no abbrev)
  gearless    → gls
  geared      → grd (but always add specs: "grd 2x30t")

DELETE FROM VOCABULARY — never use these words in offers:
  currently, suitable, opportunity, reference, documentation,
  certificate, unfortunately, furthermore, additionally,
  I would like, I wanted to, I am writing, please find,
  draw your attention, for your consideration, at your convenience,
  do not hesitate, looking forward, I hope this finds you
```

---

## PART B — REGISTER LOCK RULES

```
SYNTAX RULES — how broker English works:
  1. Drop articles (a, an, the) — ALWAYS
     ❌ "the vessel is at the port"  →  ✅ "vsl at port"
  2. Drop pronouns where obvious
     ❌ "I have the following"  →  ✅ "hv foll"
  3. Drop "is/are/was" where possible
     ❌ "she is completing at ravenna"  →  ✅ "she compl ravenna"
  4. Max 12 words per sentence. Longer → split or cut.
  5. No compound sentences. No semicolons. One thought per line.
  6. No hedging: never "maybe", "perhaps", "might", "I think"
  7. No apologizing: never "a bit late", "tight", "unfortunately"

SELF-CHECK — if you wrote a grammatically perfect English sentence
with subject + verb + object, no abbreviations, and >8 words:
  → REWRITE IT. It doesn't sound like a broker.

DENSITY TEST — count words before cargo block:
  STRONG: (1-2 sentences)
  ASK: (2-3 sentences)
  BLOCKED: (1-2 sentences)
  the amount of sentances is given just as estimation. its not a must to use the same amount of sentances.
  If over these limits → you're writing too much. Cut.
```

---

## PART C — BAD→GOOD PAIRS (from real system output + new scenarios)

### C1. BLOCKED — Gear mismatch

```
BAD (real output, manisa amelia, Score 60, BLOCKED):
  "sending fyi for mv manisa amelia. hv foll bb marmara>red sea,
   but cargo needs vsl gear both ends. sending in case you have
   another suitable vsl or plans change. any update on her current
   position and schedule?"

PROBLEMS:
  ❌ "cargo needs vsl gear both ends" — EXPOSES block reason as statement
  ❌ "another suitable vsl" — corporate language
  ❌ "or plans change" — vague filler
  ❌ "any update on her current position and schedule" — formal questionnaire
  ❌ 3 sentences before cargo block — too long for BLOCKED

GOOD (how broker writes):
  "hv foll nice cargo hvr she is gless, yes? sub chrtr to accept
   gless bss. what is her opening area\dts?"

WHY: Gear framed as QUESTION to owner ("she is gless, yes?").
  Limitation placed on CHARTERER side ("sub chrtr to accept").
  Asks operational question about opening. Active deal-making.
```

### C2. BLOCKED — Timing (vessel too early)

```
BAD (generated example of common AI pattern):
  "see she opens varna early march. my layday is 15-25 feb so
   unfortunately she is a bit too early for this cargo, however
   sending fyi in case the schedule moves up or if laycan extends.
   any update on her current program?"

PROBLEMS:
  ❌ "unfortunately" — apologizing
  ❌ "a bit too early" — telling owner what he can see himself
  ❌ "however" (full word) — use "hvr" or better, don't hedge
  ❌ "in case the schedule moves up or if laycan extends" — 2 hedges in 1 sentence
  ❌ "any update on her current program" — formal
  ❌ 40+ words before cargo — way too much for BLOCKED

GOOD:
  "hv foll cgo for mv lady speranza. she opens varna beg march,
   right? cargo l/c 15/25 feb. what frt will she need for foll
   cgo so i can check her dts workability?"

WHY: Confirms position as question. States laycan as fact, no apology.
  Asks freight aim = tool to probe owner's interest and check flexibility.
  Broker takes work on HIS side ("i can check her dts workability").
```

### C3. BLOCKED — Vessel too far (distance)

```
BAD (real output, misk m, Score 45, BLOCKED):
  "i see she's currently at djen djen — sending foll fyi in case
   her schedule changes and she heads towards the black sea after
   discharge. i know it's a long ballast but she's familiar with
   the region and owners like the area."

PROBLEMS:
  ❌ "currently" — AI marker word
  ❌ "i know it's a long ballast" — apologizing + stating obvious
  ❌ "she's familiar with the region" — exposing CRM port history
  ❌ "owners like the area" — exposing CRM preferences (Tier 1 violation)
  ❌ 2 paragraphs of context before cargo block

GOOD:
  "i see mv lady speranza is at djen djen. do you plan to take
   her bsea? hv foll cgo. lmk pls"

WHY: States position. Asks THE right question ("plan to take her bsea?").
  No distance commentary. No CRM exposure. Direct.
```

### C4. BLOCKED — Timing (vessel too late, geopolitical)

```
BAD (real output, algrace, Score 62.6, BLOCKED):
  "see mv algrace is currently laden and heading to tuapse.
   my dates are 15-18 apr, so she might be a bit tight after
   discharge and ballasting to odessa. sending fyi in case her
   schedule changes or for future reference."

PROBLEMS:
  ❌ "currently laden and heading" — owner KNOWS his ship status
  ❌ "might be a bit tight" — apologizing
  ❌ "after discharge and ballasting to odessa" — calculating route for owner
  ❌ "for future reference" — corporate email language
  ❌ "see mv algrace is currently" — 3 AI markers in one sentence
  ❌ MISSED THE REAL ISSUE: RU→UA is impossible due to war

GOOD:
  "see mv algrace is sailing tuapse. understand she cant go direct
   to ukr after ru thus sending you cargo just for good order,
   no pressure at all"

WHY: Names the REAL reason (geopolitical, not timing). Honest and direct.
  "just for good order, no pressure" = professional FYI framing.
  NOTE: AI must understand geopolitical context (RU→UA = impossible).
```

### C5. BLOCKED — Intake too small (P5)

```
BAD (common AI pattern):
  "sending fyi for yr lady. cargo is 12k mt but her intake is
   approximately 8,500 mt which might be a bit tight for full
   parcel. however sending in case charterers accept part cargo
   or if you have another vessel in your fleet."

PROBLEMS:
  ❌ "her intake is approximately 8,500 mt" — Tier 1 violation, exposing vessel limits
  ❌ "might be a bit tight" — apologizing
  ❌ "charterers accept part cargo" — speculating about charterer flexibility
  ❌ "another vessel in your fleet" — formal

GOOD:
  "hv foll nice cargo hvr hv issue with intake :)
   is her intake abt 8.5k mts? would you like this cargo?
   if yes what is frt aim so i can talk with trader if 8.5k
   can be accepted"

WHY: Honest about intake issue, with smile :). Intake as QUESTION.
  Asks freight aim = probes interest. Takes work on HIS side
  ("i can talk with trader"). Active deal-making, not passive FYI.
```

### C6. BLOCKED — Owner restriction ("no POC")

```
NOTE: This is NOT a bad pattern. Both versions below are acceptable.

VERSION 1 (more formal):
  "i know POC is not usually your preferred area, but sending
   fyi in case anything has changed on your side."

VERSION 2 (more telex):
  "remember u dont prefer poc hvr sending foll cgo just incase
   smth chng"

WHY BOTH ARE OK: "i know you don't prefer X" sounds like memory from
  working together (Tier 2 — acceptable). This is NOT CRM exposure.
  It's natural broker-to-owner communication.
```

### C7. ASK (65%) — Vessel completing nearby, heading unknown

```
BAD (common AI pattern):
  "I see that your vessel has recently completed discharge at
   Ravenna and appears to be available for next employment.
   I have the following cargo which would be a good fit for her
   given her current position in the Adriatic."

PROBLEMS:
  ❌ "I see that your vessel" — formal, not telex
  ❌ "appears to be available for next employment" — corporate
  ❌ "which would be a good fit" — AI phrase, telling owner what fits HIS vessel
  ❌ "given her current position in the Adriatic" — explaining geography
  ❌ Every word is full English, zero abbreviations

GOOD:
  "see she compl ravenna — heading anywhere specific?
   hv foll nice grain trip for her if she opens BS side"

WHY: AIS observation as question. Hook = "nice grain trip" (not analysis).
  "if she opens BS side" = conditional without apologizing.

NOTE: Ask about position ONLY when open_area is not available.
  If open_area exists → it's a FACT, don't re-ask.
  If only AIS destination/current area → it's ASSUMPTION, ask to confirm.
```

### C8. ASK (72%) — Forward cargo, vessel has current voyage

```
BAD:
  "I noticed she is currently loading at Novorossiysk. Given that
   our laycan is mid-April, she should have time to complete this
   voyage and ballast to Constanta for loading. The timing works
   well if she maintains her average speed."

PROBLEMS:
  ❌ "I noticed" — formal
  ❌ "currently loading" — "currently" banned
  ❌ "she should have time to complete..." — calculating voyage for owner
  ❌ "the timing works well" — AI assessment phrase
  ❌ "maintains her average speed" — showing you looked up speed data
  ❌ 3 sentences of analysis, zero hook

GOOD:
  "she ldg novo now — what is her next open port?
   hv foll grain ctza>emed, mid april laycan"

WHY: AIS as question. Asks about program (forward cargo = ask about future).
  States laycan. Owner decides if timing fits. No calculations.
```

### C9. STRONG (85%) — Zero ballast, vessel at load port

```
BAD:
  "Great news! I can see that your vessel is currently at
   Constanta which is exactly where the cargo is loading. This
   means zero ballast for her which is commercially very attractive.
   The stowage factor of 48 suits her grain capacity perfectly
   and the timing aligns well with the laycan."

PROBLEMS:
  ❌ "Great news!" — AI enthusiasm
  ❌ "currently at Constanta which is exactly where" — stating obvious
  ❌ "zero ballast...commercially very attractive" — analyzing for owner
  ❌ "stowage factor of 48 suits her grain capacity perfectly" — Tier 1 CRM exposure
  ❌ "timing aligns well with the laycan" — more AI assessment
  ❌ 4 sentences of analysis. Owner needs CARGO, not analysis.

GOOD:
  "see she at ctza — hv foll wheat same port ldg. free for next?"

WHY: Position = cargo location (the hook is obvious, don't explain it).
  One short question. Owner sees the zero ballast himself.

RULE: NEVER ask about cubic/SF/grain capacity. Owner knows his vessel.
  Even at high SF (56+) — just propose cargo. Owner will say if doesn't fit.
```

### C10. STRONG (88%) — Geared vessel for BB cargo, nearby

```
BAD:
  "I have a very suitable breakbulk cargo for your vessel. She is
   geared which is required for this cargo at both load and discharge
   ports. Her position at Nemrut Bay means a very short ballast
   to the loading port of Izmir."

PROBLEMS:
  ❌ "very suitable" — AI assessment
  ❌ "which is required for this cargo" — telling owner what cargo needs
  ❌ "Her position at Nemrut Bay means a very short ballast" — analysis
  ❌ Full sentences, zero abbreviations, zero telex style

GOOD:
  "see she at nemrut — hv foll bb izmir>haifa, grd 2x30t suits
   this pipes cgo. short hop. compl there and free?"

WHY: States gear WITH specs (2x30t). "short hop" = concrete value.
  "compl there and free?" = operational question.
  Maximum density. No filler.
```

### C11. ASK (67%) — Owner has regional concern ("ukr: expensive")

```
BAD:
  "I understand that Ukraine is typically expensive for your
   operations, however I have a nice cargo loading Chornomorsk
   with discharge in Mersin which would put her into East Med
   where you prefer to trade."

PROBLEMS:
  ❌ "I understand that Ukraine is typically expensive" — CRM exposure
  ❌ "where you prefer to trade" — CRM exposure of preferences
  ❌ "however" full word
  ❌ Formal sentence structure throughout

GOOD:
  "see she compl ravenna — heading anywhere specific?
   hv foll corn chorno>mersin, short trip and disch puts her
   into emed. is ukr possible for onw?"

WHY: Question about plans. Hook = "disch puts her into emed" (real benefit).
  "is ukr possible?" = natural question without exposing CRM data.

NOTE: Ask about position ONLY when open_area is not available.
```

### C12. BLOCKED — Multiple blocks (gear + timing + distance)

```
BAD:
  "Unfortunately there are several challenges with this cargo for
   your vessel. She is gearless while the cargo requires cranes,
   she is currently far from the loading area, and the timing does
   not align well. However, sending for your information in case
   circumstances change."

PROBLEMS:
  ❌ "Unfortunately" — apologizing
  ❌ Lists all block reasons — exposing analysis
  ❌ "several challenges" — corporate language
  ❌ "circumstances change" — formal
  ❌ NEVER list block reasons. Just FYI.

GOOD:
  "incase of interest"

WHY: Multiple blocks = maximum brevity. 3 words + cargo block.
  No explanations whatsoever.
```

### C13. ASK (70%) — Vessel heading AWAY from cargo area

```
BAD:
  "I notice she is currently heading towards Durres which is in
   the opposite direction from the Black Sea loading area.
   However if she returns to the Black Sea region after completing
   her current voyage, I have a suitable cargo for her."

PROBLEMS:
  ❌ "opposite direction from the Black Sea" — geography lesson
  ❌ "However if she returns" — speculating on vessel's plans
  ❌ "suitable cargo" — AI assessment
  ❌ Full formal English

GOOD:
  "see she heading durres — coming back BS after?
   hv foll wheat ctza>alex if she opens BS side"

WHY: AIS observation + repositioning question (FIRST question per rules).
  Conditional cargo pitch. No analysis.
```

### C14. STRONG (90%) — Perfect match, everything fits

```
BAD:
  "This is an excellent match for your vessel! She is perfectly
   positioned at Derince, the intake fits the cargo quantity,
   the laycan timing is spot on, and she is geared as required.
   I strongly recommend considering this cargo."

PROBLEMS:
  ❌ "excellent match" — AI enthusiasm
  ❌ Lists every scoring criterion — sounds like estimation report
  ❌ "I strongly recommend" — broker doesn't recommend, he proposes
  ❌ "spot on" — AI filler

GOOD:
  "hv foll nice cgo for yr lady :)
   grd 2x35t suits this bb. interesting?"

WHY: Confident opener ("nice cgo for yr lady").
  Gear mentioned because BB (relevant info).
  "interesting?" = asks about interest (not "free?" — at 90% vessel is likely available).
```

---

## PART D — GOLD STANDARD EXAMPLES BY SITUATION

These are template examples to store in vector DB, tagged by situation.
Pull 2-3 relevant ones into each offer request.

### D1. Tag: `blocked_gear` — Vessel gearless, cargo needs gear

```
EMAIL:
  Mustafa

  merhaba ! hope yr week started well !

  re: mv deniz star

  hv foll nice cargo hvr she is gless, yes? sub chrtr to accept
  gless bss. what is her opening area\dts?

  4k steel coils in bb
  derince\ashdod
  2x be
  10-15 apr
  2.5

MESSENGER:
  merhaba Mustafa !
  deniz star — hv nice bb hvr she gless right? sub chrtr accept
  gless bss. opening area\dts?

  4k steel coils in bb
  derince\ashdod
  2x be
  10-15 apr
  2.5
```

### D2. Tag: `blocked_timing_early` — Vessel opens too early for laycan

```
EMAIL:
  Nikos

  kalimera ! hope all good !

  re: mv aegean wave

  hv foll cgo for yr lady. she opens varna beg march, right?
  cargo l/c 25-30 apr. what frt will she need so i can check
  her dts workability?

  8k 5% barley stw abt 50 wog
  ctza\larnaca
  1x be
  25-30 apr
  2.5 add

MESSENGER:
  kalimera Nikos !
  aegean wave — she opens varna beg march right? my l/c 25-30 apr.
  what frt aim so i can check dts workability?

  8k 5% barley stw abt 50 wog
  ctza\larnaca
  1x be
  25-30 apr
  2.5 add
```

### D3. Tag: `blocked_timing_late` — Vessel arrives after cancelling

```
EMAIL:
  Selin

  iyi günler !

  re: mv celine

  see she heads tuapse — hv foll wheat danube>italy, fyi incase
  sched changes after disch

  5k 10% bulk wheat stw abt 46 wog non-imo
  danube\italy
  1x\1x
  spot
  2.5 add

MESSENGER:
  hey Selin !
  celine heads tuapse — fyi hv wheat danube>italy incase sched
  changes after

  5k 10% bulk wheat stw abt 46 wog non-imo
  danube\italy
  1x\1x
  spot
  2.5 add
```

### D4. Tag: `blocked_distance` — Vessel too far for her size

```
EMAIL:
  Amr, Loai

  ahlan !

  re: mv nile queen

  i see she is at djen djen. do you plan to take her bsea?
  hv foll cgo. lmk pls

  6k sugar in bags
  poc\tripoli
  1x be
  april
  2.5

MESSENGER to Amr:
  ahlan Amr !
  nile queen at djen djen — plan to take her bsea? hv foll cgo

  6k sugar in bags
  poc\tripoli
  1x be
  april
  2.5

MESSENGER to Loai:
  ahlan Loai !
  nile queen — she going bsea after? hv sugar poc>tripoli

  6k sugar in bags
  poc\tripoli
  1x be
  april
  2.5
```

### D5. Tag: `blocked_intake` — Cargo too big for vessel

```
EMAIL:
  Dimitris

  kalimera !

  re: mv small star

  hv foll nice cargo hvr hv issue with intake :)
  is her intake abt 8.5k mts? would you like this cargo?
  if yes what is frt aim so i can talk with trader if 8.5k
  can be accepted

  12k 5% corn stw abt 46 wog non-imo
  ctza\mersin
  1x be
  15-25 mar
  2.5

MESSENGER:
  kalimera Dimitris !
  small star — nice corn ctza>mersin hvr intake issue :)
  her intake abt 8.5k? interested? give me frt aim and i
  check with trader

  12k 5% corn stw abt 46 wog non-imo
  ctza\mersin
  1x be
  15-25 mar
  2.5
```

### D6. Tag: `blocked_owner_restriction` — Owner said "no" to region

```
EMAIL:
  Yiannis

  kalimera !

  re: mv olympic trader

  remember u dont prefer poc hvr sending foll cgo just incase
  smth chng

  5k 10% urea stw abt 45
  poc\libya
  0.8x\1x
  april
  2.5

MESSENGER:
  kalimera Yiannis !
  olympic trader — i know poc not yr area hvr fyi incase smth chng

  5k 10% urea stw abt 45
  poc\libya
  0.8x\1x
  april
  2.5
```

### D7. Tag: `strong_zero_ballast` — Vessel AT load port

```
EMAIL:
  Cengizhan

  merhaba ! hope the week is going smooth !

  re: mv star lady

  see she at ctza — hv foll wheat same port ldg. free for next?

  5k 5% wheat stw abt 48 wog non-imo
  ctza\alexandria
  1x be
  20-28 feb
  2.5 add

MESSENGER:
  merhaba Cengizhan !
  star lady at ctza — hv wheat same port ldg. free?

  5k 5% wheat stw abt 48 wog non-imo
  ctza\alexandria
  1x be
  20-28 feb
  2.5 add
```

### D8. Tag: `strong_geared_bb` — Geared vessel for BB cargo, close

```
EMAIL:
  Dimitris

  kalimera ! hope yr wednesday is going well !

  re: mv pacific trader

  see she at nemrut — hv foll bb izmir>haifa, grd 2x30t suits
  this pipes cgo. short hop. compl there and free?

  2k steel pipes in bb
  izmir\haifa
  2x be
  1-10 mar
  3.75 ttl

MESSENGER:
  kalimera Dimitris !
  pacific trader — bb pipes izmir>haifa, grd suits. free?

  2k steel pipes in bb
  izmir\haifa
  2x be
  1-10 mar
  3.75 ttl
```

### D9. Tag: `strong_short_hop` — Vessel nearby, excellent fit

```
EMAIL:
  Mehmet

  merhaba !

  re: mv karadeniz

  hv foll nice cgo for yr lady :) short hop novo>mersin.
  interesting?

  8k 5% sbm stw abt 56 wog non-imo
  novo\mersin
  1x be
  spot
  2.5

MESSENGER:
  merhaba Mehmet !
  karadeniz — nice sbm novo>mersin :) interesting?

  8k 5% sbm stw abt 56 wog non-imo
  novo\mersin
  1x be
  spot
  2.5
```

### D10. Tag: `strong_grain_capacity_match` — Grain cargo, good fit

```
EMAIL:
  Alexandros

  kalimera ! hope friday treating you well !

  re: mv agios nikolaos

  hv foll wheat ctza>alex, nice trip for her. eta abt 10th?

  10k 5% wheat stw abt 52 wog non-imo
  ctza\alexandria
  1x be
  8-15 mar
  2.5 add

MESSENGER:
  kalimera Alexandros !
  agios nikolaos — wheat ctza>alex. eta abt 10th?

  10k 5% wheat stw abt 52 wog non-imo
  ctza\alexandria
  1x be
  8-15 mar
  2.5 add
```

### D11. Tag: `ask_completing_nearby` — Vessel completing, ask about plans

```
EMAIL:
  Yiannis

  kalimera ! hope monday is treating you well !

  re: mv aegean star

  see she compl ravenna — heading anywhere specific?
  hv foll corn chorno>mersin, short trip and disch puts her
  into emed. is ukr possible for onw?

  10k 5% corn stw abt 46 wog non-imo
  chornomorsk\mersin
  1x be
  25 feb - 5 mar
  2.5 add

MESSENGER:
  kalimera Yiannis !
  aegean star — compl ravenna? heading where?
  hv corn chorno>mersin, nice short trip. ukr ok for onw?

  10k 5% corn stw abt 46 wog non-imo
  chornomorsk\mersin
  1x be
  25 feb - 5 mar
  2.5 add
```

### D12. Tag: `ask_heading_away` — Vessel going opposite direction

```
EMAIL:
  Alexandros

  kalimera !

  re: mv poseidon

  see she heading durres — coming back BS after?
  hv foll wheat ctza>alex if she opens BS side

  12k 5% wheat stw abt 48 wog non-imo
  ctza\alexandria
  1x be
  1-10 mar
  2.5 add

MESSENGER:
  kalimera Alexandros !
  poseidon heading durres — back to BS after?
  hv wheat ctza>alex if she opens tt side

  12k 5% wheat stw abt 48 wog non-imo
  ctza\alexandria
  1x be
  1-10 mar
  2.5 add
```

### D13. Tag: `ask_forward_cargo` — Forward dates, vessel mid-voyage

```
EMAIL:
  Osman

  merhaba ! hope all good yr side !

  re: mv black pearl

  she ldg novo now — what is her next open port?
  hv foll cgo in mind fr her:

  8k 5% barley stw abt 50 wog non-imo
  ctza\mersin
  1x be
  10-20 apr
  2.5

MESSENGER:
  merhaba Osman !
  black pearl ldg novo — where she opens after?
  hv foll cgo in mind fr her:

  8k 5% barley stw abt 50 wog non-imo
  ctza\mersin
  1x be
  10-20 apr
  2.5
```

### D14. Tag: `ask_vessel_at_shipyard` — Vessel at known DD location

```
EMAIL:
  Stavros

  kalimera !

  re: mv maria s

  see she at tuzla — dd or disch? hv foll cgo in mind fr her
  if she opens soon:

  5k 10% bulk urea stw abt 45 wog non-imo
  ctza\tripoli
  0.8x\1x
  1-10 mar
  2.5

MESSENGER:
  kalimera Stavros !
  maria s at tuzla — dd or disch?
  hv foll cgo in mind fr her:

  5k 10% bulk urea stw abt 45 wog non-imo
  ctza\tripoli
  0.8x\1x
  1-10 mar
  2.5
```

### D15. Tag: `ask_documentary_cross_country` — Vessel must travel to different country

```
EMAIL:
  Ahmed

  ahlan ! hope all good !

  re: mv sahara

  see she disch alexandria — do you plan to take her bsea?
  hv foll cgo in mind fr her:

  6k 10% corn stw abt 46 wog non-imo
  galati\libya
  1x\1x
  1-10 apr
  2.5

MESSENGER:
  ahlan Ahmed !
  sahara disch alex — plan to take her bsea?
  hv foll cgo in mind fr her:

  6k 10% corn stw abt 46 wog non-imo
  galati\libya
  1x\1x
  1-10 apr
  2.5
```

### D16. Tag: `inform_low_score` — Score 50-59%, FYI with soft question

```
EMAIL:
  Giorgos

  kalispera !

  re: mv hellas pride

  sending fyi — hv foll cgo incase of interest.
  where does she open after current trip?

  10k 5% wheat stw abt 48 wog non-imo
  ctza\piraeus
  1x be
  1-10 apr
  2.5 add

MESSENGER:
  kalispera Giorgos !
  hellas pride — fyi hv foll cgo incase of interest.
  where she opens after?

  10k 5% wheat stw abt 48 wog non-imo
  ctza\piraeus
  1x be
  1-10 apr
  2.5 add
```

### D17. Tag: `ask_ais_name_mismatch` — DB name ≠ AIS name

```
EMAIL:
  Osman

  merhaba !

  re: mv karadeniz

  btw AIS shows her as DEVA — did you sell yr lady or renamed?
  either way hv foll nice grain for her area

  8k 5% corn stw abt 46 wog non-imo
  ctza\mersin
  1x be
  spot
  2.5

MESSENGER:
  merhaba Osman !
  karadeniz — AIS says DEVA, sold or renamed?
  hv corn ctza>mersin if still yrs

  8k 5% corn stw abt 46 wog non-imo
  ctza\mersin
  1x be
  spot
  2.5
```

### D18. Tag: `strong_vessel_idle` — Vessel sitting at anchor, waiting

```
EMAIL:
  Panagiotis

  kalimera ! hope yr thursday going well !

  re: mv olympos

  see she sitting burgas anch — hv foll nice cgo fr her,
  gets her moving :) interesting?

  10k 5% corn stw abt 46 wog non-imo
  ctza\alexandria
  1x be
  spot
  2.5 add

MESSENGER:
  kalimera Panagiotis !
  olympos at burgas anch — hv nice corn ctza>alex, gets her
  earning :) interested?

  10k 5% corn stw abt 46 wog non-imo
  ctza\alexandria
  1x be
  spot
  2.5 add
```

### D19. Tag: `ask_very_early` — Vessel opens 14+ days before laycan

```
EMAIL:
  Mehmet

  merhaba !

  re: mv anatolia

  where is yr lady now? hv foll forward cgo mid april —
  she can fix smth short first then load this after

  6k 5% urea stw abt 45 wog
  poc\mersin
  0.8x\1x
  10-20 apr
  2.5

MESSENGER:
  merhaba Mehmet !
  anatolia — where is she? hv forward urea poc>mersin mid april.
  can she do smth short b4?

  6k 5% urea stw abt 45 wog
  poc\mersin
  0.8x\1x
  10-20 apr
  2.5
```

### D20. Tag: `blocked_multiple` — 2+ block reasons

```
EMAIL:
  Dimitris

  re: mv small trader

  incase of interest

  8k steel billets
  ctza\ravenna
  2x be
  1-10 mar
  3.75 ttl

MESSENGER:
  Dimitris, small trader — incase of interest

  8k steel billets
  ctza\ravenna
  2x be
  1-10 mar
  3.75 ttl
```

---

## PART E — SITUATION→TAG MAPPING (for vector DB retrieval)

When generating an offer, select 2-3 examples by matching:

```
1. Match by INTENT:
   intent = "blocked_info" → pull from blocked_* tags
   intent = "strong"       → pull from strong_* tags
   intent = "ask"          → pull from ask_* tags
   intent = "inform"       → pull from inform_* tags

2. Match by BLOCK REASON (if blocked):
   P8 gear blocked         → blocked_gear
   P7 timing early         → blocked_timing_early
   P7 timing late          → blocked_timing_late
   P1 distance             → blocked_distance
   P5 intake               → blocked_intake
   P2 owner restriction    → blocked_owner_restriction
   Multiple blocks         → blocked_multiple

3. Match by SITUATION (if not blocked):
   vessel at load port     → strong_zero_ballast
   vessel geared + bb cgo  → strong_geared_bb
   vessel nearby           → strong_short_hop
   grain cargo good fit    → strong_grain_capacity_match
   vessel idle/at anchor   → strong_vessel_idle
   vessel completing       → ask_completing_nearby
   vessel heading away     → ask_heading_away
   forward cargo           → ask_forward_cargo
   vessel at shipyard      → ask_vessel_at_shipyard
   cross-country travel    → ask_documentary_cross_country
   vessel very early       → ask_very_early
   AIS name ≠ DB name      → ask_ais_name_mismatch
   low score 50-59%        → inform_low_score

4. Match by CULTURE (for greeting style):
   Turkish owner           → merhaba / iyi günler
   Greek owner             → kalimera / kalispera
   Egyptian/Arabic owner   → ahlan
   Russian/Ukrainian owner → privet (if familiar)
```

---

## PART F — KEY RULES LEARNED FROM REVIEW

```
1. NEVER ASK ABOUT CUBIC/SF/GRAIN CAPACITY
   Owner knows his vessel. Even at high SF (56+) — just propose cargo.
   Owner will say if capacity doesn't fit.

2. GEAR BLOCK → QUESTION + CHARTERER SIDE
   Don't hide gear issue. Frame as question ("she is gless, yes?")
   and put limitation on charterer ("sub chrtr to accept gless bss").

3. TIMING BLOCK → ASK FREIGHT AIM
   Don't apologize for timing mismatch. Ask "what frt will she need
   so i can check her dts workability?" = probes interest + takes
   work on broker's side.

4. DISTANCE BLOCK → ASK ABOUT PLANS
   Don't comment on distance. Ask "do you plan to take her bsea?"
   Direct question about owner's plans.

5. INTAKE BLOCK → HONEST + FREIGHT AIM
   Be honest about intake issue (with :) smile). Ask freight aim
   so broker can talk with trader about accepting smaller quantity.

6. GEOPOLITICAL CONTEXT
   AI must understand real-world constraints (RU→UA impossible due to war).
   Don't calculate ETA when the route itself is blocked.

7. OPEN_AREA vs AIS
   open_area exists → FACT, don't re-ask position
   only AIS data → ASSUMPTION, ask to confirm

8. ACTIVE DEAL-MAKING, NOT PASSIVE FYI
   Even blocked vessels — broker looks for ways to make deal work.
   Freight aim = tool to probe interest and check flexibility.
   "i can check/talk with trader" = takes work on broker's side.

9. "interesting?" vs "free?"
   High score (90%+) → "interesting?" (vessel likely available, ask about interest)
   Lower score → "free?" (need to confirm availability first)

10. MESSENGER — ALWAYS ADDRESS BY NAME
    Even ultra-short blocked messages: "Dimitris, small trader — incase of interest"
```

---

## SUMMARY — What to do with this pack

1. **BAD→GOOD pairs (Part C)** → Add to prompt Section 1B (after existing pairs)
2. **Register lock (Part B)** → Add as new Section 1C in prompt
3. **Abbreviations (Part A)** → Replace current abbreviation handling in prompt
4. **Gold standards (Part D)** → Store in vector DB with tags from Part E
5. **Retrieval logic (Part E)** → Implement in offer generation pipeline
6. **Key rules (Part F)** → Add to prompt as new section

ALL EXAMPLES REVIEWED AND APPROVED BY VITALY — 2026-04-09.
