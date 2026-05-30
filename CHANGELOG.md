# Changelog

## [0.1.0-beta] — 2026-05-30

### Added
- Initial release: Jeff the Land Shark, Hide & Seek (swim) build
- Full damage formula engine with five-factor multiplier stack
- Total Output sub-formula with sqrt diminishing returns per slot
- Level-21 multiplicative crit/precision node
- Amortised crit + precision expected-value term
- Three self-test cases validating engine within 1 % of reference values
- Per-ability base damage inputs (Ram, Splish Splash, Aqua Burst, Ult Swallow, Hyper Sprint, Healing Bubble)
- Special gear toggles: West Coast Bionic Fin, Skuld's Mouthwash
- Full 21-level talent tree with 60-point budget validation
- Gear slots: Weapon × 2, Armor, Accessory, Exclusive × 2 — each with base stat + 5 modifier rows (keyword label matching)
- Realm tier inputs (Output Boost, Health, KO drop rates)
- Condition toggles: Boss, Melee <5 m, Full/High HP
- Vulnerability % and four external multipliers
- Marginal optimizer: ranks stats by % damage gain per +1 unit
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
