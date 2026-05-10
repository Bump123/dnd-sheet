# PDF Import — Oversigt

Viser præcis hvilke felter der importeres fra PDF, og hvilke der forbliver tomme/standard.

---

## WotC AcroForm (officielt D&D 2024 PDF)

Detekteres automatisk når PDF'en indeholder felter som `ClassLevel` og `STR`.

| Felt i appen       | PDF-feltnavn(e)                                      | Importeres |
|--------------------|------------------------------------------------------|------------|
| Navn               | `CharacterName`                                      | ✅         |
| Klasse             | `ClassLevel` (fx "Bard 3" → "Bard") eller `Class`   | ✅         |
| Level              | `ClassLevel` (fx "Bard 3" → 3) eller `Level`        | ✅         |
| Baggrund           | `Background`                                         | ✅         |
| Race               | `Race`                                               | ✅         |
| Subklasse          | `Subclass`                                           | ✅         |
| Alignment          | `Alignment`                                          | ✅         |
| STR/DEX/CON/INT/WIS/CHA | `STR`, `DEX`, `CON`, `INT`, `WIS`, `CHA`       | ✅         |
| AC                 | `AC`                                                 | ✅         |
| Speed              | `Speed`                                              | ✅         |
| Skill proficiencies | `ProfAcrobatics`, `ProfDeception` osv. (18 felter) | ✅         |
| Skill expertise    | `ExpAcrobatics`, `ExpDeception` osv.                 | ✅         |
| Features & Traits  | `Features`                                           | ✅         |
| Languages & Profs  | `ProfsLangs`                                         | ✅         |
| Personality/Ideals/Bonds/Flaws | `PersonalityTraits`, `Ideals`, `Bonds`, `Flaws` | ✅ (slås sammen) |
| Backstory          | `Backstory`                                          | ✅         |
| Coins (PP/GP/EP/SP/CP) | `PP`, `GP`, `EP`, `SP`, `CP`                    | ✅         |
| Våben (op til 3)   | `Wpn1_Name`, `Wpn1_AtkBonus`, `Wpn1_Damage` osv.    | ✅         |
| Spells             | —                                                    | ❌ (for kompleks struktur i PDF) |
| Inventory/gear     | `Equipment` (fritekst, én linje per item)            | ✅ (matches mod GEAR_DB — ukendte items får w:0, cost:'?') |
| Weapon Mastery     | —                                                    | ❌         |
| Spell slots (brugte) | —                                                  | ❌ (nulstilles) |
| Death saves        | —                                                    | ❌ (nulstilles) |
| Inspiration        | —                                                    | ❌ (nulstilles) |

---

## Custom PDF (generisk, text-nummererede felter)

Detekteres når PDF'en mangler `ClassLevel`/`STR` men har `Text7`.

| Felt i appen       | PDF-feltnavn       | Importeres |
|--------------------|--------------------|------------|
| Navn               | `Text1`            | ✅         |
| Klasse             | `Text7`            | ✅         |
| Level              | `Text11`           | ✅         |
| Baggrund           | `Text6`            | ✅         |
| Race               | `Text8`            | ✅         |
| Alignment          | `Text100`          | ✅         |
| AC                 | `Text13`           | ✅         |
| Speed              | `Text27`           | ✅         |
| STR                | `Text63`           | ✅         |
| INT                | `Text64`           | ✅         |
| WIS                | `Text65`           | ✅         |
| DEX                | `Text66`           | ✅         |
| CON                | `Text67`           | ✅         |
| CHA                | `Text68`           | ✅         |
| Features           | `Text57`, `Text58` | ✅ (slås sammen) |
| Traits             | `Text55`           | ✅         |
| Languages          | `Text98`           | ✅         |
| Backstory          | `Text97`           | ✅         |
| PP / GP / EP / SP / CP | `Text271`, `Text269`, `Text268`, `Text267`, `Text270` | ✅ |
| Skill proficiencies | —                 | ❌ (checkboxes ikke pålideligt læsbare) |
| Skill expertise    | —                  | ❌         |
| Våben              | —                  | ❌ (ikke ekstrakterbart fra custom format) |
| Spells             | —                  | ❌         |
| Inventory/gear     | —                  | ❌         |

---

## Hvad sker der ved import

1. **Alle felter nulstilles** inden PDF læses — så tomme PDF-felter vises som tomme i appen
2. PDF-formatet detekteres automatisk (WotC vs. custom)
3. Fundne felter udfyldes; felter der ikke findes i PDF'en forbliver tomme
4. Spells og inventory skal tilføjes manuelt efter import
