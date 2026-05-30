# Blood Hunt — Damage Calculator

A local, offline damage calculator and build optimizer for the **Marvel Rivals PvE "Blood Hunt"** mode.

> **Beta — v0.1.0** | Currently covers **Jeff the Land Shark** (Hide & Seek / swim build).

---

## Quick start

1. Download or clone this repo.
2. Open **`blood-hunt-calculator.html`** directly in any modern browser — no server, no install, no internet required.
3. Fill in your base damage values from the practice range and your gear stats.
4. Everything recalculates live.

---

## Features

| Panel | What it does |
|---|---|
| **Base Damage** | Per-ability inputs (Ram, Splish Splash, Aqua Burst, Ult, Sprint, Bubble). Enter the actual in-game base hit with no gear, then add the flat weapon bonus on top. Special gear toggles (West Coast Bionic Fin, Skuld's Mouthwash) apply on top. |
| **Offensive Stats** | Total Damage %, Boss/Melee/Full-HP conditional bonuses, gear modifier rows. |
| **Condition Toggles** | Live on/off for Boss, Target <5 m, Target Full/High HP. Conditionals add into the Total Damage bucket, not as separate multipliers. |
| **Total Output** | Base output (default 278 = max arcana + talents + realm) plus per-slot raw output rolls. Uses the sqrt diminishing-returns sub-formula. |
| **Crit / Precision** | CR, CD, PR, PD inputs. Level-21 multiplicative node selector (×1.5 CD or ×2.0 PD). Warns if CR+PR > 100 %. |
| **Vulnerability & External Mults** | Vulnerability % (e.g. Kingpin post-jump +100 %) and four labeled external multipliers (default 1.0). |
| **Talent Tree** | Full 21-level tree with point counter (60-point budget). Validates over/under budget. Exclusive nodes are radio buttons. |
| **Gear** | Weapon × 2, Armor, Accessory, Exclusive × 2. Each slot has a base stat and five modifier rows (type a label + value; the engine keyword-matches the label to the correct formula bucket). |
| **Realm** | Four account-level realm tiers. |
| **Results** | Large avg-damage-per-hit display, raw hit value, per-multiplier breakdown table, approximate boss note. |
| **Optimizer** | Ranks every formula stat by marginal % gain per +1 unit at your current build state. Includes CR/CD and PR/PD balance ratio hints. |
| **Reference** | Source priority order and per-slot preferred modifier lists. |
| **Save / Load / Compare** | Export build to JSON, import from JSON, load a second build for side-by-side comparison. |
| **Self-Test** | Runs three reference cases on load and shows PASS/FAIL with error %. |

---

## Damage formula

```
raw_hit_damage =
    base_weapon_damage
  × (1 + (total_damage_pct + active_conditional_pct) / 100)
  × (1 + total_output / 100)
  × (1 + vulnerability_pct / 100)
  × (1 + primary_weapon_extra_damage_pct / 100)
  × (1 + ability_extra_damage_pct / 100)

avg_damage_per_hit =
    raw_hit_damage
  × (1 + CR/100 × (CD/100 − 1) + PR/100 × (PD/100 − 1))
  × ext_mult_1 × ext_mult_2 × ext_mult_3 × ext_mult_4
```

### Total Output sub-formula

```
total_output = base_output + 11.5 × (√N_weapon + √N_amulet + √N_exclusive)
```

`base_output` defaults to **278** (max arcana + talents + realm, no gear output rolls).

### Level-21 multiplicative node

Mutually exclusive. Applied before the crit/precision term:
- **×1.5 Crit Damage** — multiplies CD by 1.5
- **×2.0 Precision Damage** — multiplies PD by 2.0

> There is a code comment in `computeDamage` marking where to switch this to "multiply final hit" if playtesting shows that model instead.

### Self-test cases (must pass within 1 %)

| Case | Base | Total Dmg | Cond | TO | CR | CD | PR | PD | Expected avg |
|---|---|---|---|---|---|---|---|---|---|
| Precision | 800 | 604 | 1998 | 278 | 24 | 682 | 9 | 10628 | ~979,596 |
| Crit | 800 | 963 | 1168 | 518 | 50 | 2512 | 2 | 975 | ~1,459,829 |
| Mixed | 800 | 425 | 432 | 718 | 40 | 1344 | 6 | 7234 | ~645,434 |

---

## How to measure base damage

1. Enter the practice range.
2. Equip **no gear**.
3. Hit an enemy with the ability you want to measure.
4. Record the damage number — this is your **actual base hit**.
5. Add the flat stat from your **weapon gear item** (listed on the item card as a flat +N damage).
6. Enter the sum in the corresponding Base Damage field.

> Base damage is **not** derivable from percentages. It has internal per-weapon scaling that must be measured in-game.

---

## Gear modifier labels

The modifier rows accept free-text labels. The engine keyword-matches them to formula buckets:

| Type what | Maps to |
|---|---|
| `total dmg` / `total damage` | Total Damage % |
| `crit rate` / `cr` | Crit Rate |
| `crit dmg` / `cd` | Crit Damage |
| `prec rate` / `pr` | Precision Rate |
| `prec dmg` / `pd` | Precision Damage |
| `total output` / `to` | Total Output (weapon slot bucket) |
| `boss` | Boss conditional % |
| `melee` | Melee conditional % |
| `full hp` / `fullhp` | Full-HP conditional % |
| `primary wpn` / `primary weapon` | Primary Weapon Extra Damage % |
| `ability extra` | Ability Extra Damage % |
| `vuln` | Vulnerability % |

Labels that don't match are stored but ignored by the formula (safe for defensive stats like HP, Armor, Block).

---

## Special gear

| Item | Ability | Effect | Verify note |
|---|---|---|---|
| **West Coast Bionic Fin** | Ram (collision) | +10,390 % of Hide-and-Seek base collision damage added to ram. | Verify whether this adds to base before the multiplier stack or acts as its own separate multiplier — see code comment in `hero.specialGear.wcbf`. |
| **Skuld's Mouthwash** | Splish Splash | +3,000 % damage as a separate damage-over-time hit. | Verify whether the DoT inherits the full multiplier stack or only the base — see code comment in the Mouthwash branch of `recalc`. |

---

## Adding heroes / builds (data-only)

Everything hero-specific lives in the `HERO_DATA` object at the top of the file. To add a new hero:

1. Copy the `jeff` block and rename the key (e.g. `HERO_DATA.squirrelgirl`).
2. Fill in abilities, offensiveStats, talent, gear, realm, priorityOrder, slotPrefs.
3. Add a hero-selector UI element that calls `recalc()` after switching `HERO_DATA[activeHero]`.

The formula engine (`computeDamage`, `computeTotalOutput`, `computeOptimizer`) takes a plain `stats` object and never references `HERO_DATA` — no engine changes required.

To add the **spit build** for Jeff: add a second entry under `HERO_DATA` (e.g. `jeff_spit`) with `Kraken Blaster` as the primary weapon and Splish Splash as the primary-weapon ability instead of ram.

---

## Future / known gaps (beta)

| Item | Status |
|---|---|
| Affix-tier optimizer | Stubbed. Hook documented in code: `HERO_DATA.jeff.talentCdCoeff`. Waiting on per-tier affix tables. When available, scale CD/PD affixes as `(CD_current / coeff + affix) × coeff`. |
| West Coast Bionic Fin application order | Needs in-game verification (additive to base vs. separate multiplier). |
| Skuld's Mouthwash DoT stack | Needs in-game verification (inherits full stack or base only). |
| Level-21 node application order | Implemented as "multiply CD/PD before crit term." Needs playtesting to confirm vs. "multiply final hit." |
| Level 8 point budget | Total 60-point budget marked CONFIRM in data — verify after adding level 8 costs. |
| Level 13 ability extra | Marked CONFIRM: verify +50% ability extra is % not flat. |
| Spit build | Data block not yet added (swim build is complete). |
| Other heroes | Architecture ready; data blocks not yet written. |
| Defensive stat modeling | Defensive stats (HP, armor, dodge, block, healing) are stored and displayed but do not feed the damage formula. Survivability modeling is a future feature. |
| Mobile layout | Two-column layout is desktop-first; narrow screens may need horizontal scroll. |

---

## Browser compatibility

Tested in Chrome 124+, Firefox 126+, Edge 124+. No polyfills needed. Safari 17+ should work. Does not require any server or local storage — all state is in-memory (use Export JSON to persist a build).

---

## Contributing

This is a community tool. If you find a formula discrepancy, have measured base damage values, or have verified special gear behavior:

1. Open an issue describing what you measured and how.
2. If you have corrected data, edit the `HERO_DATA` block in a fork and submit a PR — no engine changes should be needed for data corrections.

---

## Disclaimer

This tool is a fan-made community project. Marvel Rivals and all associated characters are property of NetEase Games / Marvel. This project is not affiliated with or endorsed by NetEase or Marvel.
