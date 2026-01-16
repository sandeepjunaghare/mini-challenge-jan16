# Comparison: Thales vs Ericsson Positions on GNSS Resilient NR-NTN

**Meeting:** 3GPP RAN WG1 #123, Dallas, USA (November 17-21, 2025)
**Agenda Item:** 10.6.1
**Documents:** R1-2508465 (Thales), R1-2508843 (Ericsson)

---

## Key Philosophical Differences

### Thales: Network-Assistance Focused
- Emphasizes **proactive network support** with assistance information
- Views GNSS unavailability primarily as a **system-wide issue requiring new mechanisms**
- Willing to introduce **new closed-loop procedures** (especially for frequency control)

### Ericsson: Detection & Reuse Focused
- Emphasizes **detection mechanisms first** (knowing when GNSS is lost)
- Views GNSS loss as potentially **cell-specific OR UE-specific**
- Prefers **reusing existing NR mechanisms** with minimal new procedures

---

## Specific Position Contrasts

### 1. Network Assistance Approach

**Thales (Proposal 1):**
```
For quasi-Earth-fixed cells:
→ Provide reference location of serving cell (per cell or per beam)

For Earth-moving cells:
→ Provide common TA and common Doppler value
```

**Ericsson (Observation 3):**
```
Questions the need for network-provided reference position:
→ "Further study needed on whether network should provide
   reference position or if UE should assume beam center"
→ More cautious about adding new signaling
```

---

### 2. Detection of GNSS Loss

**Thales (Proposal 2):**
- Asks whether GNSS unavailability applies to entire cell or specific UE
- Less emphasis on detection mechanisms
- Focuses more on what to do **after** loss is known

**Ericsson (Observation 1 & Proposal 1):**
```
Strong emphasis on detection:
→ UE-assisted detection (UE determines GNSS loss)
→ Network-assisted detection (network determines GNSS loss)
→ Must detect before enabling resilient features
```
**Ericsson explicitly states:** "An NTN cell may need to simultaneously support UEs with and without temporary loss of GNSS"

---

### 3. Frequency Control

**Thales (Observation 3):**
```
Proposes NEW closed-loop frequency control:
→ "Study a closed-loop method for UL frequency compensation,
   such as having gNB signal a frequency adjustment command"
→ Notes residual frequency errors when using reference location
```

**Ericsson (Observation 5):**
```
OPPOSES new frequency control:
→ "Closed-loop signaling for frequency control is NOT needed"
→ Argues existing ±0.1 ppm requirement is sufficient
→ "Differential Doppler is within existing specifications"
```

**This is a MAJOR DISAGREEMENT.**

---

### 4. PRACH/Random Access Solutions

**Thales:**
- Analyzes PRACH tolerance with differential delays
- Focuses on **common TA approach** to reduce differential delay
- Less emphasis on PRACH procedure changes

**Ericsson (Proposal 2):**
```
Proposes PRACH enhancement mechanisms:
→ PRACH repetition (transmit same preamble multiple times)
→ PRACH diversity (transmit multiple different preambles)
→ Leverage existing Rel-18 PRACH repetition as baseline
```
**Key quote:** "Existing NR preamble formats can be reused. The second transmission is also an existing NR preamble format."

---

### 5. Security Emphasis

**Thales:**
- **Strong emphasis** on GPS spoofing threats
- Cites specific data: "1500 flights per day experiencing spoofing" (OPSGROUP 2024)
- Shows charts of spoofing incidents increasing 5x since April 2024
- **Observation 1:** "GNSS is vulnerable to jamming, spoofing..."

**Ericsson:**
- Mentions jamming/spoofing as causes but **no detailed security analysis**
- More generic treatment of GNSS unavailability causes
- Less emphasis on threat landscape

---

### 6. Evaluation Methodology

**Thales:**
```
Provides extensive evaluation tables:
→ Detailed differential RTD and frequency offset values
→ Multiple orbit scenarios (LEO-600, LEO-1200, GSO)
→ Multiple elevation angles (8° to 90°)
→ Both S-band and Ka-band results
```

**Ericsson:**
```
Comments on calculation methodology:
→ Compares Alt 1 vs Alt 2 from previous agreements
→ Notes issues with existing equations
→ More focus on methodology correctness than specific values
→ Questions which alternative should be adopted
```

---

### 7. Backward Compatibility

**Thales:**
- Proposes using existing mechanisms (SIB19) for new parameters
- Reference location concept builds on existing ephemeris signaling
- Maintains Rel-17 design philosophy with modifications

**Ericsson:**
```
More explicit about backward compatibility:
→ "Existing NR mechanisms can be reused"
→ "Existing Rel-18 PRACH repetition scheme can be baseline"
→ Questions need for new RRC signaling
```

---

### 8. Position Summary Table

| Aspect | **Thales** | **Ericsson** |
|--------|------------|--------------|
| **Philosophy** | Network-centric solutions | Minimal change, detection-first |
| **New signaling** | Willing to add (reference location, common TA/Doppler) | Prefers reusing existing |
| **Frequency control** | Proposes new closed-loop | Says NOT needed |
| **PRACH solution** | Common TA approach | Repetition/diversity |
| **Security focus** | High (detailed spoofing data) | Moderate (generic mention) |
| **Detection** | Secondary concern | Primary concern |
| **Cell vs UE-specific** | Asks the question | Explicitly addresses both cases |
| **Evaluation data** | Extensive tables with values | Methodology critique |

---

## Areas of Agreement

Both companies agree on:
- Need to support GNSS unavailability scenarios
- Importance of differential delay/Doppler evaluation
- Using cell/beam reference locations as a concept
- Maintaining backward compatibility goal
- Focus on Scenario 1 (complete GNSS unavailability) first
- Applicable to both NGSO and GSO constellations
- Support for both FR1 and FR2 frequency bands

---

## Strategic Implications

### Thales' Position Would Lead To:
- More network signaling and assistance
- New closed-loop mechanisms
- More operator control over resilience
- Potentially more complex specifications
- Greater emphasis on security considerations

### Ericsson's Position Would Lead To:
- Lighter specification impact
- More UE autonomy in detection
- Reuse of existing procedures
- Potentially faster standardization
- Focus on robustness through diversity/repetition

---

## Key Technical Debates

### 1. Closed-Loop Frequency Control
- **Most contentious issue**
- Thales wants it; Ericsson opposes it
- Will require working group consensus

### 2. Detection Mechanisms
- Ericsson prioritizes detection before solutions
- Thales focuses on solutions assuming detection is handled
- Need to resolve detection approach first

### 3. PRACH Enhancement Strategy
- Thales: Reduce differential delay via common TA
- Ericsson: Handle residual errors via repetition/diversity
- Both approaches may be complementary

### 4. Granularity of Assistance
- Cell-level vs beam-level reference information
- Network-determined vs UE-determined GNSS loss
- Signaling overhead vs accuracy tradeoffs

---

## Likely Working Group Outcomes

### Areas Requiring Consensus:
1. **Detection mechanisms** - UE-assisted, network-assisted, or both
2. **Frequency control** - New closed-loop or rely on existing ±0.1 ppm
3. **PRACH procedures** - Common TA, repetition, diversity, or combination
4. **Signaling additions** - What new parameters are truly necessary

### Potential Compromise:
- Adopt detection mechanisms (Ericsson priority)
- Add reference location signaling (Thales proposal)
- Use PRACH repetition as fallback (Ericsson proposal)
- Defer closed-loop frequency control decision (study further)

---

## Document References

- **Thales:** R1-2508465, "Considerations on NR-NTN Resilience to GNSS Unavailability and Degradation"
- **Ericsson:** R1-2508843, "On NR-NTN GNSS resilience"
- **Study Item:** RP-251863, "Study on GNSS resilient NR-NTN operation"
- **Previous agreements:** RAN1#122, RAN1#122bis
