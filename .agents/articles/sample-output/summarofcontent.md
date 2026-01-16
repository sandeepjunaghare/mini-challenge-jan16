# Summary of .docx Files Content

## Overall Context

All documents are **contributions to 3GPP RAN WG1 Meeting #123** held in Dallas, USA (November 17-21, 2025) under **Agenda Item 10.6.1**. They address a **Release 20 Study Item on GNSS Resilient NR-NTN (Non-Terrestrial Network) Operation**.

## Key Objectives

The study focuses on scenarios where GNSS information in UE with GNSS capability may be:
1. **Temporarily unavailable** (primary focus)
2. Available but with GNSS position accuracy degradation
3. Available but with increased GNSS measurement period for power saving

**Goals:**
- Assess impact on initial access and connected mode procedures
- Minimize physical layer procedures impact
- Avoid physical layer channel/signal changes
- Maintain backward compatibility with legacy NTN capable UEs
- Applicable to NGSO and GSO constellations, below and above 10 GHz, transparent and regenerative satellites

## Study Scenarios

**Scenario 1:** UE cannot rely on GNSS for timing and frequency compensation on the service link

**Scenario 2:** Previously acquired GNSS-based position exists, but UE hasn't received GNSS information for time duration T, causing GNSS accuracy degradation

## Key Technical Topics

### 1. **Differential Delay and Doppler Evaluations**

Companies provided extensive evaluation results for:
- **Differential one-way delays**
- **Differential one-way Doppler shifts/frequency offsets**

Evaluated across:
- **Satellite orbits:** LEO-600 (600 km altitude), LEO-1200 (1200 km), GSO (35,786 km)
- **Frequency bands:** FR1 (2 GHz, S-band, Ku-band 14 GHz), FR2 (Ka-band 20/30 GHz)
- **Elevation angles:** 30°, 50°, 70°, 90° for LEO; 12.5° minimum for GSO
- **UE types:** Handheld (L/S band), VSAT (Ku/Ka)
- **UE speeds:** 3 km/h, 120 km/h, 1500 km/h
- **Uncertainty areas:**
  - Case a: Area served by the cell or beam
  - Case b: Circle of radius X km (1, 5, 10, 25 km, etc.)

### 2. **PRACH Performance Analysis**

Documents discuss PRACH (Physical Random Access Channel) tolerance when GNSS is unavailable:
- Impact of differential RTT (Round Trip Time)
- Frequency offset limitations
- Cyclic prefix duration considerations
- Timing error (Te) derivation methods
- Detection performance degradation analysis

### 3. **Real-World GNSS Degradation Causes**

Documents identify scenarios causing GNSS unavailability:
- **Physical blockage:** Tunnels, underground spaces, building interiors
- **Urban canyons:** Tall buildings causing multipath and signal blockage
- **Mountainous terrain:** Ridgelines and cliffs obstructing line-of-sight
- **Intentional interference:** GNSS jamming and spoofing
- **Natural phenomena:** Solar storm activity
- **Operational:** Reduced GNSS measurement for power saving

### 4. **Potential Solutions Discussed**

Companies proposed various approaches:
- **Network-provided assistance:**
  - Cell-level or beam-level common timing advance (TA)
  - Cell-level or beam-level common Doppler information
  - Satellite ephemeris broadcasting
  - Reference location information

- **UE-based mechanisms:**
  - Using last known GNSS position with validity duration
  - Doppler/delay tracking in connected mode
  - Compensation based on previous measurements

- **Hybrid approaches:**
  - Combining network assistance with UE tracking capabilities
  - Fallback mechanisms when GNSS degrades

### 5. **Contributing Companies**

Documents from: Futurewei, Eutelsat, Spreadtrum/UNISOC, vivo, CMCC, ETRI, Sony, MediaTek, Panasonic, Ericsson, Fraunhofer, iDirect, Toyota/ITC, Samsung, Thales, Tejas Networks, China Telecom, and others.

## Evaluation Methodology

RAN1#122bis agreements established:
- Standardized formulas for calculating differential delays and Doppler
- Specific geometries (within orbital plane, at 90° to orbital plane)
- Beam pointing assumptions (quasi-Earth-fixed beams)
- UE altitude error considerations
- Specific evaluation cases and parameters

## Next Steps

The study includes a checkpoint at **RAN#112 (June 2026)** to decide whether to:
- Proceed with normative work
- Extend the study item
- Postpone normative work to Release 21

---

**Summary:** This comprehensive summary covers all 45 .docx files. The documents represent a coordinated industry effort to ensure NR-NTN systems can maintain connectivity even when GNSS signals are unavailable or degraded, which is critical for reliable satellite-based communication services.
