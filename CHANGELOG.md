# Changelog

## [0.3.0] — 2026-05-31

### Added

- **Item Comparison panel** — define two hypothetical items for any slot (Weapon / Armor / Accessory / Exclusive), see a live per-ability damage table showing Current / Item A / Item B with Δ% and Best column; updates automatically when the main build changes; gear-driven mode performs a true slot swap, manual mode adds on top with a warning banner; S-tier fill and Clear buttons per item; grade badges on all modifier rows
- **Base Damage Estimator panel** — reverse-calculates the correct Base Damage field value from an observed in-game hit; accepts ability and hit type (Normal / Crit / Prec), divides out all known multipliers, strips weapon flat and ability flat bonuses (Bubbling Impact, Tsunami, etc.) and the WCBF factor; "Apply" button writes the result directly to the Base Damage field; full divisor breakdown shown for verification

### Changed

- **Gear model** — each slot is now a single equipped item (one Weapon, one Armor, one Accessory, one Exclusive) with an item selector for slots that offer multiple choices (Deep Sea Propellor vs Kraken Blaster; WCBF vs Mouthwash); corrects the incorrect two-simultaneously-equipped-items model
- **Gear modifier rows** — replaced free-text keyword labels with stat dropdowns filtered to each equipment type's eligible pool; grade badges (F–D–C–B–A–S) shown per modifier value; S-tier fill button; Clear button; live contribution summary per item
- **Master stat pool** — new `STAT_POOL` data block with F-tier and S-tier ranges for all 19 stats; grade interpolation is linear (real cutoffs noted as unconfirmed)
- **Jeff character enhancements** — 7 talent-affix gear modifiers (Forward Frenzy, Un-Shark-Stoppable, Bubbling Impact, Big Fish Frenzy, Hyper Sprint Bubble, Deep Breath, Hangry) added as selectable gear modifier stats on Weapon / Accessory / Exclusive
- **Ability-aware damage engine** — formula now includes `(ability_base + flat_bonuses) × (1 + ability_pct/100)` before the global multiplier stack; talent lines and gear enhancements route into per-ability flat/pct pools via `abilityBonusMap`
- **Total Output** — "Base Output" renamed to "Arcana Output" (default 0); talent Output Amplification (levels 3 and 11) and realm output tiers are now added automatically on top, eliminating the double-counting that occurred when the old 278 default already included talent contributions
- **Optimizer** — uses S-tier roll as the increment instead of +1 unit for realistic marginal analysis; CD/PD increments scaled by talent-tree coefficient (1.5× or 2.0× from the level-21 node); shows both per-S-tier and per-unit gain; combinatorial optimizer hook documented in code
- **Stats source toggle** — "Gear-driven" mode (default) computes all stats from talent + gear mods + realm and makes aggregate fields read-only; "Manual" mode restores direct aggregate-field editing for quick what-if testing
- **Results panel** — added all-abilities summary table at top (click any row to select it for detailed breakdown); Mouthwash DoT shown as a child row under Splish Splash
- **Condition toggles** — added Big Fish Frenzy toggle (10+ enemies swallowed, condition for Ult Swallow)
- **Base Damage info box** — updated to reflect that weapon flat is added automatically and the Estimator is available as an alternative

### Architecture

- New `STAT_POOL`, `JEFF_ENHANCEMENTS`, and `GEAR_ELIGIBLE_POOLS` data blocks; all hero-specific talent-to-ability routing lives in `abilityBonusMap`; formula engine (`computeDamage`, `computeTotalOutput`, `computeOptimizer`) is unchanged and stateless
- Self-tests (Precision 0.98%, Crit 0.000%, Mixed 0.48%) all still pass within 1%

---

## [0.1.0-beta] — 2026-05-30

### Added

- Initial release: Jeff the Land Shark, Hide & Seek (swim) build
- Full damage formula engine with five-factor multiplier stack
- Total Output sub-formula with sqrt diminishing returns per slot
- Level-21 multiplicative crit/precision node
- Amortised crit + precision expected-value term
- Three self-test cases validating engine within 1% of reference values
- Per-ability base damage inputs (Ram, Splish Splash, Aqua Burst, Ult Swallow, Hyper Sprint, Healing Bubble)
- Special gear toggles: West Coast Bionic Fin, Skuld's Mouthwash
- Full 21-level talent tree with 60-point budget validation
- Gear slots: Weapon, Armor, Accessory, Exclusive — each with base stat and 5 modifier rows
- Realm tier inputs (Output Boost, Health, KO drop rates)
- Condition toggles: Boss, Melee <5 m, Full/High HP
- Vulnerability % and four external multipliers
- Marginal optimizer: ranks stats by % damage gain per unit
- CR/CD and PR/PD balance ratio hints in optimizer
- Source priority reference panel and per-slot preferred modifier lists
- Save/load via JSON export/import
- Side-by-side build compare
- Per-multiplier breakdown table in results panel

### Known gaps / beta caveats

- Special gear application order (West Coast Bionic Fin, Skuld's Mouthwash) needs in-game verification
- Level-21 node application order (CD/PD vs. final hit) needs playtesting confirmation
- Level 8 talent point budget needs confirmation
- Level 13 ability extra confirmed as % not flat — needs in-game verification
- Affix-tier optimizer stubbed; waiting on per-tier affix tables
- Spit build data block not yet added
- No other heroes implemented yet
- Defensive stats stored but not used in any calculation
