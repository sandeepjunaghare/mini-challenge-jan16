# Companies' Positions on Positioning-Based Approaches for GNSS Resilient NR-NTN

**Meeting:** 3GPP RAN WG1 #123, Dallas, USA (November 17-21, 2025)
**Agenda Item:** 10.6.1
**Topic:** GNSS Resilient NR-NTN Operation

---

## Companies SUPPORTING Positioning Approaches

### 1. **Ofinno (R1-2509131)** - STRONG SUPPORT

**Position:** Proposes DL-TDOA positioning as "Solution 2A"

**Proposal 5:** "Study the feasibility of UE-based DL-TDOA for determining the location position during initial access procedure across all the relevant working groups (RAN1, RAN2, and RAN4)"

**Approach:**
- Single/multi-satellite DL-TDOA based on current specifications
- UE measures PRS from multiple satellites to determine position via RSTD measurements
- Acknowledges need for LMF assistance via LPP signaling

**Challenges noted:**
- Requires RAN4 to define new UE requirements for UE-based positioning
- Initial access complexity (UE needs to reach RRC_CONNECTED to request assistance data)
- Existing specifications (TS 38.133) only specify RSTD measurement requirements, not UE-based positioning requirements

**Observations:**
- **Observation 8:** "The LPP specification (TS 37.355) supports signaling and procedures related to the UE-based DL-TDOA for determining the UE position"
- **Observation 9:** "For determining positioning based on DL-TDOA, the UE needs to send a request to LMF via LPP to receive DL-TDOA assistance data from the LMF"
- **Observation 10:** "Current specifications (e.g., TS 38.133) specify only UE requirements related to the RSTD measurements and don't specify UE requirements related to UE-based DL-TDOA"

---

### 2. **Airbus (R1-2508872)** - STRONG SUPPORT

**Position:** Advocates multi-satellite NTN RAT-dependent positioning

**Key statement:** "RAT-dependent positioning as a complementary method for GNSS resilient NR-NTN operation"

**Observation 1:** "Coarse UE localization with RAT-dependent positioning can be performed in NR-NTN without any specification enhancement and used in combination with other solutions for GNSS resilient NR-NTN operation"

**Evidence provided:**
- Simulation results for LEO-1200 constellation at FR1-NTN band
- Horizontal positioning accuracy at 95%
- DL-TDOA method using DL-PRS over 8.64 MHz bandwidth
- Handheld terminal in line-of-sight (LOS) conditions
- Includes tropospheric and ionospheric ranging errors

**Technical details:**
- **Observation 4:** "A minimum of 3 satellites is sufficient to perform horizontal DL-TDOA positioning, e.g. by using a barometric pressure sensor"
- **Observation 2:** "For the example LEO constellation with minimum user elevation of 20°, the visibility of at least 3 satellites is ensured at any user latitude"
- **Observation 3:** "For the example LEO constellation with minimum user elevation of 30°, the visibility of at least 3 satellites significantly depends on the user latitude"

**Method:** UE-based DL-TDOA positioning as described in TS 38.305, using DL-PRS specified in TS 38.211

---

### 3. **IMU (R1-2509017)** - MODERATE SUPPORT

**Position:** Proposes single-satellite positioning

**Observation 1:** "Single-satellite positioning can be considered for GNSS-resilient NR-NTN using time-separated measurements within a pass"

**Approach:**
- Measurements at different time instants form "virtual anchors" along satellite path
- Practical for single-satellite visibility scenarios
- Positioning based on measurements at different time instants during satellite pass

**Key considerations:**
- Time interval between measurement points must be large enough (pass-dependent) for qualified geometry
- RTT/TOA values change little with short intervals, preventing reliable position determination
- Excessively large intervals increase access delay if UE waits to collect them before random access

**Trade-off:** Balances measurement interval (longer = better geometry, but increases access delay)

---

## Companies OPPOSING Positioning Approaches

### 1. **Qualcomm (R1-2509228, R1-2509498)** - STRONG OPPOSITION

**Position:** DL-only positioning is NOT feasible for NTN

**Observation 12:** "Enabling DL-only UE-based positioning solutions to reduce time-frequency uncertainty using measurements from multiple satellites depends on satellite constellation characteristics"

**Observation 13:** "Enabling DL-only UE-based positioning solutions to reduce time-frequency uncertainty using measurements from single satellites is not feasible"

#### Arguments Against Multi-Satellite Positioning:

1. **Constellation dependency:** Satellite constellation density must be high enough for UE to see multiple satellites above minimum elevation angle

2. **Beam-planning challenges and resource overhead:**
   - Multiple satellites need to serve the same region
   - UE must receive DL reference signals (SSB/PRS) with sufficient power
   - Poor positioning accuracy and interference to other DL signals
   - Requires tight coordination across multiple satellites
   - Satellites moving relative to each other and serving area makes coordination challenging

3. **Standards work outside scope:**
   - Requires additional 3GPP standards work outside work item description
   - PRS muting framework needs investigation to enable PRS measurements from non-serving satellites

4. **Historical context:**
   - References Release 18 discussions on network verified UE location
   - Previous observations noted major challenges with multi-satellite based operation
   - Extra complexity for UE

#### Arguments Against Single-Satellite Positioning:

1. **Excessive latency:**
   - UE requires RSTD measurements over long time window (>24 seconds)
   - Introduces significant latency before random access
   - Not practical for initial access scenarios

2. **Measurement challenges:**
   - Requires measurements over extended time periods
   - Impractical for time-sensitive initial access procedures

**Conclusion:** Qualcomm explicitly argues both multi-satellite and single-satellite DL-only positioning are not feasible for reducing time-frequency uncertainty in NTN scenarios.

---

## Companies with NEUTRAL/MIXED Positions

### **China Telecom (R1-2509185)**
- Mentions positioning accuracy degradation concerns
- Discusses need to evaluate position changes or GNSS accuracy degradation
- Notes UE cannot determine changes in position relative to previous GNSS data during signal loss
- No strong advocacy for or against positioning solutions
- Focuses more on GNSS accuracy evaluation methods and status reporting

### **CATT (R1-2508594)**
- Brief mention of position-based solutions
- No detailed positioning proposal or analysis
- Focuses on other aspects of GNSS resilience

---

## Companies Focusing on OTHER Solutions (Not Positioning)

Most other companies prefer alternative approaches to positioning:

### Network Assistance Approaches:
- **Thales (R1-2508465):** Network-provided reference location + common TA/Doppler for service link
- **Eutelsat (R1-2508360):** Common TA approach, PRACH tolerance analysis
- **Spreadtrum/UNISOC (R1-2508385):** Network-provided cell-level or beam-level common TA and Doppler
- **CMCC (R1-2508452):** Timing/frequency compensation methods using network assistance

### PRACH Enhancement Approaches:
- **Ericsson (R1-2508843):** PRACH repetition/diversity, detection mechanisms, reuse existing Rel-18 mechanisms
- **Vivo (R1-2508429):** PRACH performance evaluation, network assistance solutions

### Evaluation Focus:
- **Futurewei (R1-2508333):** Detailed differential delay/Doppler evaluation across scenarios
- **Sony (R1-2509071):** Multiple solution comparison and evaluation
- **MediaTek (R1-2509163):** Reference location with error analysis
- **Panasonic (R1-2509296):** Common TA for initial access, analytical evaluation
- **ETRI (R1-2508970):** Real-world GNSS degradation analysis (tunnels, urban canyons, jamming)

---

## Summary Table

| **Position** | **Companies** | **Count** | **Approach** |
|-------------|--------------|-----------|--------------|
| **Strong Support** | Ofinno, Airbus | 2 | Multi-satellite DL-TDOA positioning |
| **Moderate Support** | IMU | 1 | Single-satellite positioning with time-separated measurements |
| **Strong Opposition** | Qualcomm | 1 | Both multi and single-satellite approaches infeasible |
| **Neutral/Other Focus** | All others | ~41 | Common TA, PRACH enhancement, network assistance, evaluation |

---

## Key Technical Debate: Positioning vs. Non-Positioning

### Pro-Positioning Arguments (Ofinno, Airbus, IMU):

**Benefits:**
- Can provide actual UE location (not just uncertainty area)
- Existing LPP specifications (TS 37.355) support it
- No new physical layer changes needed (uses existing DL-TDOA framework)
- Simulation results show feasibility for certain constellation configurations
- Can be used as complementary method alongside other solutions

**Technical Foundation:**
- TS 38.305 already defines UE-based DL-TDOA positioning method
- TS 38.211 specifies DL-PRS for positioning
- TS 37.355 supports necessary signaling and procedures

### Anti-Positioning Arguments (Qualcomm):

**Feasibility Concerns:**
- Constellation-dependent (not universally available across all deployments)
- High latency, especially for single-satellite approaches (>24 seconds)
- Complexity and resource overhead for multi-satellite coordination
- Beam planning challenges across non-colocated satellites

**Specification Concerns:**
- Outside work item scope (SID explicitly states: "Note: This study does not include any enhancement on positioning")
- Requires additional standards work (PRS muting framework, coordination mechanisms)
- RAN4 requirements for UE-based positioning not yet defined

**Operational Concerns:**
- Interference management between satellites
- Tight coordination needed across moving satellites
- Satellite constellation characteristics vary significantly

### The Fundamental Tension:

**Scope Interpretation:**
- SID explicitly excludes positioning enhancements
- Proponents argue they're using EXISTING positioning capabilities without enhancement
- Opponents argue any positioning solution requires additional work beyond current scope
- Question remains: Does "no enhancement on positioning" mean existing positioning can be used?

---

## Analysis and Implications

### Statistical Summary:
- **Only 3 out of ~45 companies** (6.7%) explicitly support positioning approaches
- **1 company** strongly opposes positioning solutions
- **~41 companies** (91%) focus on alternative solutions

### Interpretation:
Positioning-based approaches are **minority proposals** in the working group. The majority of companies prefer:
1. Network-assisted solutions (common TA, reference locations)
2. PRACH enhancements (repetition, diversity)
3. Hybrid approaches combining network assistance with UE tracking

### Likely Working Group Outcome:
Given the distribution of positions:
- Positioning solutions unlikely to become primary approach
- May be studied as optional/complementary method
- Scope interpretation debate needs resolution first
- Focus will likely remain on non-positioning solutions (common TA, PRACH enhancements)

### Key Questions for Working Group:
1. Does using existing positioning capabilities violate the "no positioning enhancement" constraint?
2. Should positioning be studied as complementary method even if not primary solution?
3. Are simulation results from Airbus sufficient to justify further study?
4. Do Qualcomm's feasibility concerns apply to all constellation configurations?

---

## Document References

**Supporting Positioning:**
- R1-2509131 (Ofinno): "Discussion on NR-NTN GNSS resilience"
- R1-2508872 (Airbus): "Discussion on NR-NTN GNSS resilience"
- R1-2509017 (IMU): "Discussion on NR-NTN GNSS resilience"

**Opposing Positioning:**
- R1-2509228 (Qualcomm): "Discussion on GNSS-Resilient NTN"
- R1-2509498 (Qualcomm): "Discussion on GNSS-Resilient NTN"

**Relevant Specifications:**
- TS 38.305: Positioning methods and procedures
- TS 38.211: Physical layer specifications (DL-PRS)
- TS 37.355: LPP specification (positioning protocol)
- TS 38.133: Requirements for support of radio resource management

**Study Item:**
- RP-251863: "Study on GNSS resilient NR-NTN operation"
