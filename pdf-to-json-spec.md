# PDF-to-JSON — Spec

## Formål

Konverter en udfyldt D&D 2024 PDF-karakterark til `character.json` som direkte kan loades i appen. Spilleren slipper for at genudfylde alt manuelt.

---

## Placering i appen

Ny knap i karakterarkets header ved siden af "Load JSON":

```html
<label class="save-btn" style="cursor:pointer">
  📄 Import PDF
  <input type="file" accept=".pdf" onchange="importPDF(event)" style="display:none">
</label>
```

Ingen separat side — konverteringen sker inline, og resultatet populates direkte i det åbne ark (samme flow som "Load JSON").

---

## Teknisk tilgang

**PDF.js** (Mozilla, kører 100% client-side — ingen server, virker på GitHub Pages).

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.0.379/pdf.min.mjs" type="module"></script>
```

To strategier anvendes i rækkefølge:

### 1. AcroForm-felter (fillable PDF)
PDF.js kan læse AcroForm-feltnavne og -værdier direkte fra fillable PDFs (fx det officielle WotC-ark):

```js
const pdf = await pdfjsLib.getDocument({data: arrayBuffer}).promise;
const fields = await pdf.getFieldObjects();
// fields er et objekt: { "CharacterName": [{value: "Maxwin"}], ... }
```

### 2. Tekst-extraction (scanned/flad PDF)
Hvis ingen AcroForm-felter findes, bruges tekstudtræk per side. Positional heuristics bruges til at identificere felter ud fra nærliggende label-tekst.

```js
const page = await pdf.getPage(1);
const content = await page.getTextContent();
// content.items: [{str, transform: [,,,,x,y]}, ...]
```

Strategi 1 foretrækkes — strategi 2 er fallback og vil give partielle resultater.

---

## Feltmapping

### AcroForm-feltnavne → JSON-nøgler

Officielle WotC D&D 2024 PDF-feltnavne (verificeres ved første kørsel):

| PDF-feltnavn              | JSON-nøgle          | Type      | Transformation                        |
|---------------------------|---------------------|-----------|---------------------------------------|
| `CharacterName`           | `charName`          | string    | direkte                               |
| `ClassLevel`              | `cls` + `level`     | string    | split på mellemrum: "Bard 3" → cls="Bard", level=3 |
| `Background`              | `charBg`            | string    | direkte                               |
| `Race`                    | `charRace`          | string    | direkte                               |
| `Alignment`               | `charAlign`         | string    | direkte                               |
| `Subclass`                | `charSub`           | string    | direkte (hvis feltet eksisterer)      |
| `STR`                     | `scores.STR`        | int       | `parseInt`                            |
| `DEX`                     | `scores.DEX`        | int       | `parseInt`                            |
| `CON`                     | `scores.CON`        | int       | `parseInt`                            |
| `INT`                     | `scores.INT`        | int       | `parseInt`                            |
| `WIS`                     | `scores.WIS`        | int       | `parseInt`                            |
| `CHA`                     | `scores.CHA`        | int       | `parseInt`                            |
| `AC`                      | `ac`                | int       | `parseInt`                            |
| `Speed`                   | `speed`             | int       | `parseInt`                            |
| `HPMax`                   | ignoreres           | —         | appen beregner HP selv                |
| `PersonalityTraits`       | `traits` (prefix)   | string    | sættes øverst i traits-textarea       |
| `Ideals`                  | `traits` (del)      | string    | appended                              |
| `Bonds`                   | `traits` (del)      | string    | appended                              |
| `Flaws`                   | `traits` (del)      | string    | appended                              |
| `Backstory`               | `backstory`         | string    | direkte                               |
| `ProfsLangs`              | `langs`             | string    | direkte                               |
| `Features`                | `features`          | string    | direkte                               |
| `CP`, `SP`, `EP`, `GP`, `PP` | `coins.*`       | int       | `parseInt`                            |

### Skill-proficiencies
PDF-arket har checkboxes per skill (fx `ProfAcrobatics`). Mappes til:

```js
// Hvis checked → push til profSkills
// Expertise-checkbox (hvis den eksisterer) → push til expSkills
const SKILL_FIELDS = {
  'ProfAcrobatics':    'Acrobatics',
  'ProfAnimalHandl':   'Animal Handling',
  'ProfArcana':        'Arcana',
  'ProfAthletics':     'Athletics',
  'ProfDeception':     'Deception',
  'ProfHistory':       'History',
  'ProfInsight':       'Insight',
  'ProfIntimidation':  'Intimidation',
  'ProfInvestigation': 'Investigation',
  'ProfMedicine':      'Medicine',
  'ProfNature':        'Nature',
  'ProfPerception':    'Perception',
  'ProfPerformance':   'Performance',
  'ProfPersuasion':    'Persuasion',
  'ProfReligion':      'Religion',
  'ProfSleightOfHand': 'Sleight of Hand',
  'ProfStealth':       'Stealth',
  'ProfSurvival':      'Survival',
};
```

### Våben
PDF-arket har typisk 3 våben-rækker med felterne `Wpn_Name`, `Wpn_AtkBonus`, `Wpn_Damage`:

```js
for (let i = 1; i <= 3; i++) {
  const name = fields[`Wpn${i}_Name`]?.[0]?.value;
  if (name) weapons.push({
    name,
    atk:  fields[`Wpn${i}_AtkBonus`]?.[0]?.value ?? '',
    dmg:  fields[`Wpn${i}_Damage`]?.[0]?.value ?? '',
    type: '', mastery: '—'
  });
}
```

---

## Implementering — `importPDF(event)`

```js
async function importPDF(e) {
  const file = e.target.files[0];
  if (!file) return;
  const buf = await file.arrayBuffer();

  // PDF.js skal loades som modul — se init-sektion
  const pdf = await pdfjsLib.getDocument({data: buf}).promise;
  const fieldObjs = await pdf.getFieldObjects().catch(() => null);

  if (!fieldObjs || Object.keys(fieldObjs).length === 0) {
    alert('Ingen udfyldbare felter fundet i PDF. Kun fillable PDFs understøttes fuldt ud.');
    return;
  }

  const get = (key) => fieldObjs[key]?.[0]?.value ?? '';
  const getInt = (key) => parseInt(get(key)) || 0;

  // Parse klasse + niveau
  const clsRaw = get('ClassLevel').trim();        // fx "Bard 3"
  const clsMatch = clsRaw.match(/^(\D+?)\s*(\d+)$/);
  const cls   = clsMatch ? clsMatch[1].trim() : (clsRaw || C.cls);
  const level = clsMatch ? parseInt(clsMatch[2]) : C.level;

  // Skills
  const profSkills = [], expSkills = [];
  for (const [field, skill] of Object.entries(SKILL_FIELDS)) {
    if (get(field) === 'Yes' || get(field) === 'true') profSkills.push(skill);
    if (get('Exp' + field.slice(4)) === 'Yes') expSkills.push(skill);
  }

  // Traits sammensættes
  const traitParts = [
    get('PersonalityTraits'), get('Ideals'), get('Bonds'), get('Flaws')
  ].filter(Boolean);

  // Bygger nyt C-objekt (beholder state der ikke er i PDF)
  const imported = {
    ...C,
    charName:  get('CharacterName') || C.charName,
    cls, level,
    charBg:    get('Background')    || C.charBg,
    charRace:  get('Race')          || C.charRace,
    charSub:   get('Subclass')      || C.charSub,
    charAlign: get('Alignment')     || C.charAlign,
    scores: {
      STR: getInt('STR') || C.scores.STR,
      DEX: getInt('DEX') || C.scores.DEX,
      CON: getInt('CON') || C.scores.CON,
      INT: getInt('INT') || C.scores.INT,
      WIS: getInt('WIS') || C.scores.WIS,
      CHA: getInt('CHA') || C.scores.CHA,
    },
    ac:    getInt('AC')    || C.ac,
    speed: getInt('Speed') || C.speed,
    profSkills: profSkills.length ? profSkills : C.profSkills,
    expSkills:  expSkills.length  ? expSkills  : C.expSkills,
    features:  get('Features')  || C.features,
    langs:     get('ProfsLangs')|| C.langs,
    traits:    traitParts.join('\n\n') || C.traits,
    backstory: get('Backstory') || C.backstory,
    coins: {
      pp: getInt('PP') || C.coins.pp,
      gp: getInt('GP') || C.coins.gp,
      ep: getInt('EP') || C.coins.ep,
      sp: getInt('SP') || C.coins.sp,
      cp: getInt('CP') || C.coins.cp,
    },
  };

  // Våben
  const weapons = [];
  for (let i = 1; i <= 3; i++) {
    const name = get(`Wpn${i}_Name`);
    if (name) weapons.push({ name, atk: get(`Wpn${i}_AtkBonus`), dmg: get(`Wpn${i}_Damage`), type: '', mastery: '—' });
  }
  if (weapons.length) imported.weapons = weapons;

  C = imported;
  // Sync HTML-inputs der ikke renderes af render()
  document.getElementById('charName').value  = C.charName;
  document.getElementById('classSelect').value = C.cls;
  document.getElementById('charBg').value    = C.charBg;
  document.getElementById('charRace').value  = C.charRace;
  document.getElementById('charAlign').value = C.charAlign;
  document.getElementById('featuresText').value = C.features;
  document.getElementById('langsText').value    = C.langs;
  document.getElementById('traitsText').value   = C.traits;
  document.getElementById('equipText').value    = C.backstory;
  render();
  if (C.charSub) document.getElementById('charSub').value = C.charSub;
}
```

---

## PDF.js — loading

PDF.js v4 er ES-modul. Indlæses i `<head>` som `type="module"`:

```html
<script type="module">
  import * as pdfjsLib from 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.0.379/pdf.min.mjs';
  pdfjsLib.GlobalWorkerOptions.workerSrc =
    'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.0.379/pdf.worker.min.mjs';
  window.pdfjsLib = pdfjsLib; // eksponér til ikke-modul script
</script>
```

Resten af arkets `<script>` er ikke et modul — `window.pdfjsLib` bruges som bro.

---

## Feltnavne — verifikation ved implementation

De eksakte AcroForm-feltnavne varierer mellem PDF-versioner. Ved første kørsel logges alle felter til konsollen så mapping kan justeres:

```js
console.table(
  Object.entries(fieldObjs).map(([k, v]) => ({ field: k, value: v[0]?.value }))
);
```

---

## Hvad der IKKE er i scope

- Spell-liste fra PDF (for kompleks struktur til pålidelig parsing)
- Inventory/gear fra PDF (samme årsag)
- Scannede PDFs / billede-PDFs uden AcroForm (OCR kræver server)
- Multi-page parsing udover side 1 (de fleste kritiske felter ligger der)
