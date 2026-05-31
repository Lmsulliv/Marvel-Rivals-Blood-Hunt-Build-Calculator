# Blood Hunt — Damage Calculator

A local, offline damage calculator and build optimizer for the **Marvel Rivals PvE "Blood Hunt"** mode.

> **Beta — v0.3.0** | Currently covers **Jeff the Land Shark** (Hide & Seek / swim build).

---

## Quick start

1. Download or clone this repo.
2. Open **`blood-hunt-calculator.html`** directly in any modern browser — no server, no install, no internet required.
3. Set the Gear panel to the items you have equipped (one per slot).
4. Fill in base damage values from the practice range.
5. Everything recalculates live.

---

## Features

| Panel | What it does |
| --- | --- |
| **Stats source toggle** | Switch between **Gear-driven** (gear modifier rows feed all formula stats; aggregate fields are read-only derived display) and **Manual** (type stats directly for quick what-if testing). |
| **Base Damage** | Per-ability inputs (Ram, Splish Splash, Aqua Burst, Ult, Sprint, Bubble Shockwave). Enter the actual in-game base hit with no gear, then weapon flat bonus is added automatically from the equipped weapon. West Coast Bionic Fin and Skuld's Mouthwash toggles apply on top. |
| **Offensive Stats** | Total Damage %, Boss / Melee / Full-HP conditional bonuses. In manual mode these are editable; in gear-driven mode they show values derived from gear. |
| **Condition Toggles** | Live on/off for Boss, Target <5 m, Target Full/High HP, and Big Fish Frenzy (10+ enemies swallowed). Conditionals add into the Total Damage bucket, not as separate multipliers. |
| **Total Output** | **Arcana Output** (default 0) is the arcana-only contribution you measure in-game. Talent Output Amplification (levels 3 and 11) and Realm output tiers are added on top automatically. Per-slot gear output rolls feed in via the sqrt diminishing-returns sub-formula. |
| **Crit / Precision** | CR, CD, PR, PD. Level-21 multiplicative node selector (×1.5 CD or ×2.0 PD). Warns if CR+PR > 100 %. |
| **Vulnerability & External Mults** | Vulnerability % and four labeled external multipliers (default 1.0). |
| **Talent Tree** | Full 21-level tree with 60-point budget counter and validation. Exclusive nodes are radio buttons. Talent bonuses automatically route into per-ability bonus pools (Ram %, Splish %, etc.). |
| **Gear** | One slot per type: Weapon, Armor, Accessory, Exclusive. Slots with multiple item options show a selector. Each item has a fixed base stat plus **5 modifier rows**; each row is a stat dropdown (filtered to that equipment type's eligible pool) plus a value and an **F–S grade badge**. "S-tier" button fills all 5 rows at max values. "Clear" zeroes all 5 rows. A contribution summary line shows what each item adds to the formula. |
| **Realm** | Four account-level realm tiers. |
| **Results** | All-abilities summary table (click any row to select it for the detailed breakdown), large avg-damage display, raw hit value, per-multiplier breakdown table. |
| **Optimizer** | Ranks every formula stat by marginal % gain per S-tier roll at your current build state. CD and PD increments are scaled by the talent-tree coefficient. Shows per-unit gain alongside per-S-tier gain. CR/CD and PR/PD balance ratio hints. |
| **Reference** | Source priority order and per-slot preferred modifier guidance. |
| **Save / Load / Compare** | Export build to JSON, import from JSON, load a second build for side-by-side comparison. |
| **Base Damage Estimator** | Can't unequip items? Enter an observed in-game hit, select the ability and hit type (Normal / Crit / Prec), and the panel reverses the formula to produce the correct Base Damage value. Automatically removes weapon flat, ability flat bonuses, and the WCBF factor. "Apply" button fills in the Base Damage field directly. |
| **Item Comparison** | Test two hypothetical items against your current equipped item without touching your main build. Select the slot type, define up to 5 modifiers per test item (same dropdown + grade badge system as the gear panel), and see a live per-ability table showing Current / Item A / Item B damage with Δ% and a Best column. Updates automatically when the main build changes. |
| **Self-Test** | Runs three reference cases on load and shows PASS/FAIL with error %. All three must reproduce their expected values within 1 %. |

---

## Damage formula

```text
ability_avg_damage =
    (ability_base + ability_flat_bonuses)
  × (1 + ability_specific_pct / 100)
  × (1 + (total_damage_pct + active_conditional_pct) / 100)
  × (1 + total_output / 100)
  × (1 + vulnerability_pct / 100)
  × (1 + extra_damage_pct / 100)
  × (1 + CR/100 × (CD/100 − 1) + PR/100 × (PD/100 − 1))
  × ext_mult_1 × ext_mult_2 × ext_mult_3 × ext_mult_4
```

**Definitions:**

- `ability_flat_bonuses` — flat additions specific to that ability (e.g. Bubbling Impact shockwave damage, Tsunami), added to base before any multiplier.
- `ability_specific_pct` — sum of that ability's % bonuses from talent lines and gear enhancements (e.g. Forward Frenzy for Ram, Forked Stream for Splish Splash), additive.
- `extra_damage_pct` — ability extra damage (all Jeff swim abilities are tagged "ability"; the tag is a per-ability editable field in data).

### Total Output sub-formula

```text
total_output = (arcana + talent_output_amplification + realm_output)
             + 11.5 × (√N_weapon + √N_amulet + √N_exclusive)
```

The first three terms are computed automatically from the **Arcana Output** field, the talent tree, and realm tiers. `N_weapon / N_amulet / N_exclusive` are the Total Output modifier values rolled on each equipped item (read from gear in gear-driven mode, or from the manual slot inputs).

### Level-21 multiplicative node

Mutually exclusive. Applied **before** the crit/precision term:

- **×1.5 Crit Damage** — multiplies CD by 1.5
- **×2.0 Precision Damage** — multiplies PD by 2.0

> Code comment in `computeDamage` marks where to switch to "multiply final hit" if playtesting shows that model instead.

### Talent-tree coefficient (optimizer)

When the optimizer predicts adding a CD or PD affix, it scales the raw roll by the build's talent-tree coefficient before applying it:

```text
displayed_CD_new = (displayed_CD / coeff + affix_raw) × coeff
                 → displayed CD increases by affix_raw × coeff
```

`coeff` = 1.5 if the ×1.5 Crit node is taken (for CD), 2.0 if the ×2.0 Prec node is taken (for PD), 1.0 otherwise.

> **VERIFY in-game:** confirm that adding a known CD affix shifts the displayed CD stat by `affix × 1.5` (with node) vs `affix × 1` (without).

### Self-test cases (must pass within 1 %)

All ability-specific bonuses, vulnerability, and extra-damage multipliers are set to zero in these tests, so they remain unchanged by the new formula layers.

| Case | Base | Total Dmg | Cond | TO | CR | CD | PR | PD | Expected avg |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Precision | 800 | 604 | 1998 | 278 | 24 | 682 | 9 | 10628 | ~979,596 |
| Crit | 800 | 963 | 1168 | 518 | 50 | 2512 | 2 | 975 | ~1,459,829 |
| Mixed | 800 | 425 | 432 | 718 | 40 | 1344 | 6 | 7234 | ~645,434 |

---

## How to measure base damage

### Option A — Practice range (preferred)

1. Enter the practice range.
2. Equip **no gear**.
3. Hit an enemy with the ability you want to measure.
4. Record the damage number — this is your **actual base hit**.
5. Enter it in the Base Damage field. The weapon flat bonus (e.g. +1000 Ram from Deep Sea Propellor) is added automatically by the calculator from the equipped weapon's base stat field.

> Base damage is **not** derivable from percentages. It has internal per-weapon scaling that must be measured in-game.

### Option B — Base Damage Estimator (when you can't unequip)

Blood Hunt runs are always fully geared and there is no way to unequip mid-run. If you can't reach the practice range, use the **Base Damage Estimator** panel instead:

1. Set all your stats first — fill in the Gear panel (gear-driven mode) or type stats manually so every multiplier is accurate.
2. Toggle on any **conditions that were active during the hit** you're about to use (Boss, Melee, Full-HP, etc.). This is critical — a missed condition shifts the estimate.
3. Open the **Base Damage Estimator** panel in the left column.
4. Select the ability and the hit type (Normal, Crit, or Prec). Use a **normal hit** whenever possible — it requires no guessing about crit/prec state.
5. Enter the damage number shown in-game.
6. Click **Apply to Base Damage field**.

The estimator divides the observed number by every known multiplier (ability %, Total Damage + conditionals, Total Output, vulnerability, extra damage, external multipliers), then strips out the weapon flat bonus and any ability flat bonuses (Bubbling Impact, Tsunami, etc.) that the engine adds automatically. For Ram with WCBF active, it also divides out the 104.9× WCBF factor. A breakdown line shows every divisor applied so you can verify the math.

---

## Gear system

### One slot per type

| Slot | Options |
| --- | --- |
| **Weapon** | Deep Sea Propellor (swim) or Kraken Blaster (spit) — select one |
| **Armor** | Armor |
| **Accessory** | Accessory |
| **Exclusive** | West Coast Bionic Fin or Skuld's Mouthwash — select one (or none) |

Only the equipped item's base stat and modifier rows contribute to the formula.

### Modifier rows

Each item has exactly **5 modifier rows**. Each row picks one stat from a dropdown (filtered to the equipment type's eligible pool) and a rolled value. The 5 stats on a single item must be distinct.

- **Armor** can never roll damage-affecting stats. Its pool is: HP, Armor Value, Dodge Rate, Block Rate, Health %, Block Damage Reduction, Heal Cooldown Reduction, Regen/sec, Health on Kill, Extra Heal Rune Charges.
- **Weapon, Accessory, Exclusive** can roll any stat in the master pool plus Jeff's character enhancements.

The per-slot preferred modifier lists in the reference panel are **optimizer guidance only**, not restrictions.

### Grade badges

Each modifier value is shown with an F–D–C–B–A–S badge based on where it falls between the F-tier minimum and S-tier maximum for that stat (linear interpolation — real grade cutoffs may not be perfectly linear).

### S-tier reference ranges

| Stat | F-tier | S-tier |
| --- | --- | --- |
| Total Damage Bonus % | 208 | 554 |
| Crit Rate % | 6.1 | 16.2 |
| Crit Damage % | 312 | 832 |
| Precision Rate % | 2.3 | 6.2 |
| Precision Damage % | 728 | 1942 |
| Total Output Boost | 208 | 554 |
| Boss Damage % | 311 | 830 |
| Close-Range Damage % | 311 | 830 |
| Healthy-Target Damage % | 415 | 1107 |
| Health (flat) | 554 | 1450 |
| Armor Value | 192 | 512 |
| Dodge Rate % | 3.2 | 8.5 |
| Block Rate % | 7 | 18.7 |

---

## Jeff character enhancements

These talent-affix stats can roll on Weapon, Accessory, and Exclusive modifier rows. Multiple sources of the same enhancement are additive.

| Enhancement | Ability | F-tier | S-tier | Unit |
| --- | --- | --- | --- | --- |
| Forward Frenzy | Ram | 149 | 398 | % |
| Un-Shark-Stoppable | Ram | 33 | 88 | % |
| Bubbling Impact | Bubble Shockwave | 299 | 797 | flat damage |
| Big Fish Frenzy | Ult Swallow | 26 | 69 | % |
| Hyper Sprint Bubble | Hyper Sprint Dash | 598 | 1594 | % |
| Deep Breath | — (utility) | 11 | 30 | energy/kill |
| Hangry | — (utility) | 11 | 30 | ult energy/kill |

---

## Per-ability bonus routing

Talent lines and gear enhancements that affect a specific ability feed into that ability's `abilityPct` or `abilityFlat` before the global multiplier stack. Sources per ability:

| Ability | % Sources | Flat Sources |
| --- | --- | --- |
| **Ram** | Forward Frenzy (talent ×3 tiers) + Un-Shark-Stoppable (talent ×3) + Forward Frenzy Enh (gear) + Un-Shark-Stoppable Enh (gear) | — |
| **Splish Splash** | Forked Stream (talent ×3) + Bubble Assault (talent ×3) + Bogged Down (talent ×1) | — |
| **Aqua Burst** | Reservoir (talent ×3) | — |
| **Bubble Shockwave** | — | Bubbling Impact (talent ×3) + Tsunami (talent ×3) + Bubbling Impact Enh (gear) |
| **Hyper Sprint** | Hyper Sprint Bubble Enh (gear) | — |
| **Ult Swallow** | Big Fish Frenzy (talent ×3) + Big Fish Frenzy Enh (gear) | — |

---

## Special gear

| Item | Effect | Verify note |
| --- | --- | --- |
| **West Coast Bionic Fin** | Ram base effective = `measured_base × (1 + 10390/100)`. Added to base before multiplier stack. | VERIFY: applies to measured base only (before weapon flat) or to full base including weapon flat. |
| **Skuld's Mouthwash** | Splish Splash gains +3000% damage as a separate DoT hit, shown as its own row in the results table. | VERIFY: DoT inherits the full multiplier stack (current implementation) or base only. |

---

## Stats source modes

| Mode | Behavior |
| --- | --- |
| **Gear-driven** (default) | Talent tree + equipped gear modifier rows + realm feed all formula stats. CR/CD/PR/PD and the offensive stat fields update to show derived values and become read-only. Change stats by changing gear, talent, or realm. |
| **Manual** | Type stats directly into the aggregate input fields. Gear modifier rows are shown with grade badges but do not add to the formula. Use this for quick "what if I had +500 CD?" testing. |

---

## Item Comparison

The Item Comparison panel (right column, below Optimizer) lets you evaluate two hypothetical gear pieces against your current equipped item for the same slot without modifying your build.

### Workflow

1. Open the **Item Comparison** panel.
2. Select the **slot** you want to test (Weapon / Armor / Accessory / Exclusive). The currently equipped item is shown for reference.
3. Fill in **Item A** and **Item B**:
   - Optional label (e.g. "S-tier weapon" vs "current A-tier weapon")
   - Base stat (weapon flat damage or accessory base crit rate — the only base stats that affect the damage formula)
   - Up to 5 modifier rows, each with a stat dropdown and value. The dropdown is filtered to that slot type's eligible pool. Grade badges appear automatically.
   - **S-tier** button fills all 5 rows with the top eligible stats at their S-tier max values for instant best-case modelling.
   - **Clear** resets all 5 rows and the base stat.
4. The **results table** updates live and shows avg damage per ability for Current / Item A / Item B with a Δ% column and a colour-coded Best column.

### How the swap works

**Gear-driven mode (default):** The current equipped item's contributions (modifier stats and base stat where relevant) are subtracted from the live stats, then the test item's contributions are added. This is a true slot swap — the rest of your build is unchanged.

**Manual mode:** The test item's contributions are added on top of your manually entered stats. A warning banner makes this distinction clear. Switch to Gear-driven for an accurate head-to-head.

### What it accounts for

- All modifier stats routed to their correct formula buckets (Total Damage, CR/CD/PR/PD, Total Output per-slot sqrt, conditional bonuses, Jeff ability-specific enhancements)
- Weapon flat damage difference between test items (deltaA / deltaB applied per-ability)
- Accessory base crit rate difference
- All currently active conditions (Boss, Melee, Full-HP) are inherited from the main build toggles
- Results update automatically whenever talent, gear, realm, or conditions change in the main build

---

## Adding heroes / builds (data-only)

Everything hero-specific lives in the `HERO_DATA` object at the top of the file. The engine (`computeDamage`, `computeTotalOutput`, `computeOptimizer`) takes a plain `stats` object and never references hero-specific structure. To add a new hero or build:

1. Copy the `jeff` block and rename the key (e.g. `HERO_DATA.squirrelgirl`).
2. Fill in `abilities`, `abilityBonusMap`, `offensiveStats`, `talent`, `gear`, `realm`, `priorityOrder`, `slotPrefs`.
3. Add the hero's enhancements to `JEFF_ENHANCEMENTS` (or create a parallel block) and update `GEAR_ELIGIBLE_POOLS` if the eligible pool differs.
4. Add a hero-selector UI element that calls `recalc()` after switching the active hero.

To add the **spit build** for Jeff: add `jeff_spit` with Kraken Blaster as the weapon, route Splish Splash through the primary-weapon slot, and adjust `abilityBonusMap` accordingly.

---

## Future / known gaps (beta)

| Item | Status |
| --- | --- |
| WCBF application order | VERIFY in-game: does it scale measured base only, or (measured base + weapon flat)? Code comment in `resolveAbilityBase`. |
| Mouthwash DoT multiplier stack | VERIFY in-game: full stack or base only? Code comment in `recalc`. |
| Level-21 node application point | VERIFY: multiply CD/PD before crit term (current) vs. multiply final hit. Code comment in `computeDamage`. |
| Talent-tree CD/PD coefficient | VERIFY: confirm affix × 1.5/2.0 scaling by comparing a known roll with/without the node in-game. Code comment in `getTalentCoeff`. |
| Level 8 point budget | 60-point total — CONFIRM after verifying level 8 costs in-game. |
| Level 13 ability extra | CONFIRM: +50% ability extra is % not flat. |
| Arcana Output baseline | Find the arcana-only output value in-game (with 0 talent points in levels 3/11 and 0 realm tiers) and enter it in the Arcana Output field. Use the Base Damage Estimator on a known ability to back-check the value if needed. |
| Spit build | Data block not yet added; swim build complete. |
| Other heroes | Architecture ready; data blocks not yet written. |
| Combinatorial optimizer | Stubbed with documented hook in `computeOptimizer`. Will iterate all 4-item combinations under eligibility rules when implemented. |
| Defensive stat modeling | Defensive stats (HP, armor, dodge, block, healing) are stored in gear state and displayed in contribution summaries but do not feed the damage formula. Survivability modeling is a future feature. |
| Gear-driven vulnerability | Vulnerability % is currently manual-only (no gear stat maps to it). Future: Kingpin post-jump window or hero-specific sources. |
| Mobile layout | Two-column layout is desktop-first; narrow screens may require horizontal scroll. |

---

## Browser compatibility

Tested in Chrome 124+, Firefox 126+, Edge 124+. No polyfills needed. Safari 17+ should work. Does not require any server or local storage — all state is in-memory (use Export JSON to persist a build).

---

## Contributing

This is a community tool. If you find a formula discrepancy, have measured base damage values, or have verified any of the "VERIFY" items above:

1. Open an issue describing what you measured and how.
2. For data corrections (stat ranges, ability routing, enhancement values), edit the data blocks (`STAT_POOL`, `JEFF_ENHANCEMENTS`, `HERO_DATA`) in a fork and submit a PR — no engine changes required.
3. For verified application-order fixes, edit the relevant code comment and the small logic block it describes.

---

## Disclaimer

This tool is a fan-made community project. Marvel Rivals and all associated characters are property of NetEase Games / Marvel. This project is not affiliated with or endorsed by NetEase or Marvel.
