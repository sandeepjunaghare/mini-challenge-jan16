# Martian Landing Page Structure

## Aesthetic Direction: Retro-Futuristic Mars Colony

Bold, warm, alien. The red planet as home — welcoming yet otherworldly.

---

## Color Palette (NO PURPLE)

```css
:root {
  --mars-rust: #C1440E;
  --mars-dust: #E07A5F;
  --sunset-gold: #F2CC8F;
  --terra-orange: #FF6B35;
  --deep-canyon: #8B2500;
  --sand-beige: #D4A574;
  --night-black: #1A0A00;
  --star-white: #FAF9F6;
}
```

---

## Typography

- **Headlines:** "Orbitron" — geometric, space-age
- **Body:** "DM Sans" — clean, modern

---

## Page Sections

### Section 1: HERO — Welcome to Mars

**Background:** Deep rust gradient with floating dust particles
**Color:** `--mars-rust` to `--night-black`

```
╔══════════════════════════════════════════════════════╗
║                                                      ║
║              ☉                                       ║
║                                                      ║
║         W E L C O M E   T O   M A R S               ║
║                                                      ║
║      The Red Planet Awaits Your Arrival             ║
║                                                      ║
║              [ Explore → ]                          ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

### Section 2: THE TERRAIN — Explore the Surface

**Background:** Bright sunset gold with canyon texture
**Color:** `--sunset-gold` + `--terra-orange` accents

```
╔══════════════════════════════════════════════════════╗
║                                                      ║
║              T H E   T E R R A I N                  ║
║                                                      ║
║   ┌─────────┐   ┌─────────┐   ┌─────────┐          ║
║   │ OLYMPUS │   │ VALLES  │   │ THARSIS │          ║
║   │  MONS   │   │MARINERIS│   │  RISE   │          ║
║   │  ▲▲▲    │   │  ═══    │   │  ◆◆◆    │          ║
║   └─────────┘   └─────────┘   └─────────┘          ║
║                                                      ║
║        Discover the wonders of Mars                 ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

### Section 3: THE MISSION — Join the Colony

**Background:** Bright terra orange with geometric patterns
**Color:** `--terra-orange` + `--sand-beige`

```
╔══════════════════════════════════════════════════════╗
║                                                      ║
║            T H E   M I S S I O N                    ║
║                                                      ║
║     "Humanity's next chapter begins here."          ║
║                                                      ║
║              ┌─────────────────┐                    ║
║              │  JOIN THE CREW  │                    ║
║              └─────────────────┘                    ║
║                                                      ║
║         Be part of something extraordinary          ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

## Visual Effects

1. **Floating dust particles** — CSS animation, subtle drift
2. **Gradient transitions** — Smooth color flow between sections
3. **Hover glow** — Buttons pulse with warm ember light
4. **Staggered text reveal** — Headlines animate on load

---

## File Structure

```
index.html    — Single HTML file with embedded CSS
```

---

## Section Color Summary

| Section | Primary Color | Accent |
|---------|---------------|--------|
| Hero | Mars Rust (#C1440E) | Night Black |
| Terrain | Sunset Gold (#F2CC8F) | Terra Orange |
| Mission | Terra Orange (#FF6B35) | Sand Beige |
