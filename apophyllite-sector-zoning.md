# Apophyllite Sector-Zoning: Vugg Render Proposal

**Status:** Research complete, awaiting CI green before implementation  
**Date:** 2026-06-21  
**Author:** 🪨✍️ (research)  
**Target:** Vugg Simulator builder — habit-variant + render logic addition  
**Related:** `memory/research-apophyllite.md`, `memory/research-apophyllite-tn498.md`, TN498 microscope session

---

## The Problem

Builder handoff: a "zigzag / hourglass crystal" was requested for a new vugg render. Initial thought was augite (pyroxene, classic hourglass sector-zoning). But augite is a **magmatic phenocryst** — it forms in cooling lava, not in fluid-filled cavities. The entire vugg premise is cavity mineralization. Augite breaks the model.

**Replacement:** apophyllite, already in the catalog (`deccan_zeolite` scenario, round-2 addition), already researched, already has a `grow_apophyllite` engine entry. And — critically — apophyllite **already exhibits sector-specific optical properties in nature**.

---

## Why Apophyllite Works

### 1. It's Already in the Vugg Model

- `minerals.json` entry: `apophyllite`
- Scenario: `deccan_zeolite` (plus it can appear in late-stage hydrothermal vugs generally)
- T range: 20–150°C — perfect low-T vug setting
- pH: 7.5–10 — alkaline cavity fluids
- Late-stage propensity: high (forms AFTER zeolites, calcite, chalcedony — terminal phase of basalt vesicle filling)

### 2. Sector-Zoning is Optically Real

Apophyllite is a **classic example of anomalous birefringence** in optical mineralogy. Under crossed polars it shows patchy, sector-zoned extinction patterns that "shouldn't exist in a well-behaved tetragonal crystal."

**Mechanisms (documented, real):**

1. **Sector zoning** — different crystal faces incorporate water and K at different rates during growth → sectors with different refractive indices within the same crystal
2. **Variable hydration** — structural water content varies zone to zone → different optical properties per sector
3. **F/OH solid solution** — fluorapophyllite ↔ hydroxyapophyllite form a continuous solid solution; sectors can differ in F/OH ratio

**Source:** TN498 microscope session (2026-03-24), optical mineralogy texts, UFRGS apophyllite waveplate study (hagenlab.org, 2022)

### 3. The Sectors Are Growth-Defined

Apophyllite tetragonal habit has three principal growth sectors:

| Sector | Miller Index | Growth Properties | Optical Character |
|--------|-------------|-------------------|-------------------|
| **Prism** | {100} | Faster growth, lower water content | Higher luster, more transparent, vitreous |
| **Basal pinacoid** | {001} | Slower growth, higher water content | Pearly luster on cleavage, more hydrated |
| **Pyramidal termination** | {111} | Intermediate growth rate | Intermediate properties, steeper angle |

The color distribution also varies by sector:
- **Green** (V⁴⁺ trace, 1600 ppm in Poona/Deccan material — Rossman 1974): preferentially incorporates in faster-growing prism sectors. *Note: Cu is sometimes cited in loose tertiary sources but is not supported for Deccan apophyllite; V⁴⁺ is the documented chromophore for this locality.*
- **Clear/colorless**: basal sectors, water-dominated, less trace-element uptake
- **"Cinnamon" phantom**: hematite inclusions from specific growth episodes, visible in prism sectors

### 4. The Name Literally Means "It Flakes Off"

Greek: *apo* (away from) + *phyllon* (leaf). The {001} cleavage is perfect — heat it, rub it, breathe acid on it, and the layers separate like pages from a wet book.

This maps to a **degradation render state**: if a thermal event >200°C fires in the vugg, existing apophyllite crystals become flaky, lose transparency, and shed layers. The exfoliation is not just a property — it's the mineral's identity.

---

## What the Builder Needs to Add

This is **not a new mineral entry**. It's a **habit-variant + render-logic addition** to the existing `apophyllite` spec.

### Current `minerals.json` apophyllite entry has:

- `habit`: prismatic to tabular tetragonal
- `habit_variants`: [prismatic, tabular, "Bloody Phantom" (hematite inclusion)]
- `color_rules`: green with Cu trace, clear without
- `test_cases`: 3 declared

### What to add:

#### New Habit Variant: `sector_zoned` or `hourglass`

```json
{
  "name": "Sector-Zoned Cathedral",
  "wall_spread": 0.3,
  "void_reach": 0.7,
  "vector": "projecting",
  "trigger": {
    "slow_growth": true,
    "sector_zoning_enabled": true,
    "min_crystal_size_cm": 1.0
  },
  "sector_properties": {
    "prism_100": {
      "luster": "vitreous",
      "transparency": 0.9,
      "color_priority": "V_green",
      "fluorescence_intensity": 1.0
    },
    "basal_001": {
      "luster": "pearly",
      "transparency": 0.6,
      "color_priority": "clear",
      "fluorescence_intensity": 0.7,
      "exfoliation_prone": true
    },
    "pyramidal_111": {
      "luster": "vitreous_to_pearly",
      "transparency": 0.75,
      "color_priority": "intermediate",
      "fluorescence_intensity": 0.85
    }
  }
}
```

#### Render Logic (pseudocode)

```
IF crystal.species == apophyllite AND crystal.habit_variant == "sector_zoned"
  // Render three distinct sectors with different optical properties
  FOR each growth_sector in [prism, basal, pyramidal]
    SET material.luster = sector.luster
    SET material.transparency = sector.transparency
    SET material.color = resolve_color(sector.color_priority, fluid.trace_V)
    SET material.fluorescence = sector.fluorescence_intensity * base_fluorescence
  
  // Exfoliation state (post-thermal event >200°C)
  IF vugg.thermal_event_fired AND thermal_event.T_C > 200
    SET material.state = "exfoliated"
    SET material.transparency = 0.3  // cloudy, layers separating
    SET surface.roughness = HIGH  // flaking texture
    // Visual: layers peel back like pages, curved delamination
```

#### Color Resolution by Sector

```
FUNCTION resolve_color(priority, fluid_V_ppm)
  IF priority == "V_green" AND fluid_V_ppm > threshold
    RETURN green_apophyllite
  ELSE IF priority == "clear"
    RETURN colorless_clear
  ELSE IF priority == "intermediate"
    RETURN mix(clear, green, fluid_V_ppm / (2 * threshold))
  ELSE
    RETURN base_colorless
```

---

## Research Sources

| Source | What it provides |
|--------|-----------------|
| `memory/research-apophyllite.md` | Full species research — formation, paragenesis, fluorescence, variants |
| `memory/research-apophyllite-tn498.md` | Microscope session — optical effects, hematite inclusions, "Cinnamon" variety |
| TN498 specimen (DB) | Physical specimen with four simultaneous optical effects observed |
| UFRGS apophyllite waveplate study (hagenlab.org, 2022) | Sector structure nonuniformity, anomalous birefringence mechanisms |
| Ottens et al. 2019, MDPI Minerals | Deccan Traps paragenetic sequence — Stage I/II/III model, apophyllite as terminal phase |
| MinerShop fluorescent database | Uranyl ion activator for yellow-green fluorescence in prism sectors |

---

## Blockers

- **None for research** — all sources confirmed, sector-zoning is optically real
- **CI status**: Waiting on cold-CI verdict before any `minerals.json` edits
- **Implementation**: Requires builder to add habit-variant + render logic; no new mineral spec needed

---

## Alternative Render: Exfoliation Etch-Pit Sculpture

If the builder wants something more visually distinctive than sector-zoning, the **exfoliation property** itself can be a render mechanic:

- Crystal "grows" by etching away wrong sectors, leaving a pseudomorph of growth structure
- The vugg doesn't simulate the etching — it displays the result as a found object
- Unique to apophyllite: no other mineral in the catalog has exfoliation as a defining property

This is a **render-only variant**, not a growth mechanic. The crystal would spawn in an exfoliated state (post-thermal event) or the player could trigger exfoliation via a heat event.

---

## Notes for Builder

1. The sector-zoning is **growth-sector partitioning**, not compositional zoning like in feldspar. The mechanism is differential water/K uptake per face, not bulk chemistry change over time.
2. The "hourglass" visual in augite comes from {100} prism sectors meeting at a central line. In apophyllite, the analogous visual would be **prism sectors meeting at the c-axis** — the basal pinacoid forms the "waist" where growth slowed.
3. The pearly luster on {001} is a **real optical property** from higher water content and perfect cleavage. It should render differently from the vitreous prism faces.
4. Exfoliation is irreversible in the model — once flaky, the crystal stays flaky. This is geologically accurate: dehydration collapses the structure to amorphous material.

---

## Correction Log

| Date | Issue | Original | Corrected | Source |
|------|-------|----------|-----------|--------|
| 2026-06-21 | Chromophore | Cu²⁺ asserted as green colorant | **V⁴⁺** at 1600 ppm for Poona/Deccan | Rossman 1974, Am. Mineral. |
| 2026-06-21 | Sector-zoning framing | None (correct in original) | Builder initially called it "not real sector zoning, just dichroism"; verification confirmed anomalous birefringence + variable optic sign = **real growth-sector zoning** | Multiple optical mineralogy sources |
| 2026-06-21 | Color geometry | No direct measurement | Prism-green / basal-clear waist = **reasoned model** from fast-sector impurity-trapping principle, not measured sector map | Standard sector-zoning theory; no accessible apophyllite-specific data |

*Disagreement-as-blind-spot-detector: each side caught an error. The chromophore was mine (confabulated Cu); the sector-zoning reality was the doc's (confirmed by verification). The color geometry remains a reasoned model — plausible, mechanistically sound, but not directly measured.*

---
