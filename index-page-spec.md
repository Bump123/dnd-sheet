# Index Page — Spec

## Formål

Oversigtsside på `dnd-sheet/index.html`. Viser ét kort per karakter med live data trukket fra den gemte JSON. Fungerer som landing page på GitHub Pages.

---

## Data-strategi

Karakterarket har allerede "💾 Save JSON". Konventionen udvides:

- Hvert karakterark gemmer en **`character.json`** i sin egen mappe, fx `spiller1/karakter1/character.json`
- Karakterarket får en ekstra knap: **"📌 Publish"** — gemmer `character.json` (via download, som spilleren committer til repo)
- Index-siden forsøger `fetch('spillerX/karakterY/character.json')` for hvert kort ved sideindlæsning
- Fejler fetchen (404 / ingen fil endnu) vises et tomt placeholder-kort i stedet

Resultatet er en ren statisk løsning der virker på GitHub Pages uden server.

---

## Layout

```
┌─────────────────────────────────────────────────────┐
│           ⚔  D&D Party  ⚔                          │
│         Vælg en karakter                             │
├──────────┬──────────┬──────────┬──────────┬─────────┤
│ Spiller1 │ Spiller2 │ Spiller3 │ Spiller4 │Spiller5 │
│  [kort]  │  [kort]  │  [kort]  │  [kort]  │  [kort] │
├──────────┴──────────┴──────────┴──────────┴─────────┤
│                    DM  [kort]                        │
└─────────────────────────────────────────────────────┘
```

- `display: grid`, `repeat(auto-fill, minmax(220px, 1fr))`, max-width 960px
- DM-kortet er altid bredere (grid-column: span 2 ved >= 3 kolonner) eller placeret separat under spillerne
- Fonte: Cinzel (titel/labels) + Crimson Text (værdier) — matcher karakterarket

---

## Kort-indhold

### Indlæst (character.json fundet)

```
┌──────────────────────────┐
│ SPILLER 1                │  <- lille label (uppercase, dæmpet)
│ Maxwin                   │  <- charName (stor, fed)
│ Level 3 Bard             │  <- level + cls
│ College of Lore          │  <- charSub (vises kun hvis ikke tom/"—")
│ Human · Charlatan        │  <- charRace · charBg
│ HP 22  AC 15  Speed 30   │  <- calcHP, ac, speed
└──────────────────────────┘
```

### Placeholder (ingen JSON endnu)

```
┌──────────────────────────┐
│ SPILLER 2                │
│ —                        │  <- navn ikke sat
│ Intet ark gemt endnu     │  <- grå italic tekst
└──────────────────────────┘
```

Kortet er stadig klikbart og linker til karakterarket.

---

## Felter der vises (fra JSON)

| Felt       | Kilde i JSON           | Fallback |
|------------|------------------------|----------|
| Navn       | `charName`             | `—`      |
| Niveau     | `level`                | `1`      |
| Klasse     | `cls`                  | `—`      |
| Subclass   | `charSub`              | skjules  |
| Race       | `charRace`             | `—`      |
| Baggrund   | `charBg`               | `—`      |
| HP         | beregnet (se nedenfor) | `—`      |
| AC         | `ac`                   | `—`      |
| Speed      | `speed`                | `—`      |

**HP-beregning på index (samme formel som arket):**
```js
const hd = CLASS_HD[cls] ?? 8;
const cm = Math.floor((scores.CON - 10) / 2);
const hp = hd + cm + (level - 1) * (Math.ceil((hd + 1) / 2) + cm);
```

`CLASS_HD` er en lille konstant kun med HD-værdier (ikke hele CLASS_DATA).

---

## Farver

Matcher SVG-diagrammet og det mørke tema fra karakterarket.

| Element          | Baggrundsfarve | Kantfarve  | Tekstfarve |
|------------------|----------------|------------|------------|
| Spillerkort      | `#085041`      | `#5dcaa5`  | `#9fe1cb`  |
| DM-kort          | `#712b13`      | `#f0997b`  | `#f5c4b3`  |
| Placeholder-kort | `#2a2a2a`      | `#555`     | `#888`     |
| Side-baggrund    | `#1a1a2e`      | —          | `#e0ddd5`  |

Hover: `translateY(-3px)` + box-shadow (samme som nuværende index.html).

---

## Karakterark-ændringer (Publish-knap)

I `dnd-character-sheet.html` tilføjes ved siden af "Save JSON":

```html
<button class="save-btn" onclick="publishChar()">📌 Publish</button>
```

```js
function publishChar() {
  collectText();
  const a = document.createElement('a');
  a.href = 'data:application/json,' + encodeURIComponent(JSON.stringify(C, null, 2));
  a.download = 'character.json';   // fast filnavn — skal ligge i samme mappe
  a.click();
}
```

Spilleren placerer den downloadede `character.json` i sin `spillerX/karakterY/`-mappe og committer.

---

## Fetch-logik (index.html script)

```js
const CHARACTERS = [
  { label: 'Spiller 1', path: 'spiller1/karakter1/', type: 'spiller' },
  { label: 'Spiller 2', path: 'spiller2/karakter1/', type: 'spiller' },
  { label: 'Spiller 3', path: 'spiller3/karakter1/', type: 'spiller' },
  { label: 'Spiller 4', path: 'spiller4/karakter1/', type: 'spiller' },
  { label: 'Spiller 5', path: 'spiller5/karakter1/', type: 'spiller' },
  { label: 'DM',        path: 'dm/karakter1/',       type: 'dm'      },
];

const CLASS_HD = {
  Barbarian:12, Fighter:10, Paladin:10, Ranger:10,
  Bard:8, Cleric:8, Druid:8, Monk:8, Rogue:8, Warlock:8,
  Sorcerer:6, Wizard:6,
};

async function loadAll() {
  await Promise.all(CHARACTERS.map(async (ch) => {
    try {
      const res = await fetch(ch.path + 'character.json', { cache: 'no-cache' });
      if (!res.ok) throw new Error();
      const data = await res.json();
      renderCard(ch, data);
    } catch {
      renderCard(ch, null);
    }
  }));
}
```

`Promise.all` sikrer at alle fetch-kald kører parallelt — siden loader ikke sekventielt.

---

## Hvad der IKKE er i scope

- Redigering af kort direkte på index-siden
- Autentificering / private karakterark
- Realtidsopdatering (ingen WebSocket/polling)
- Flere karakterer per spiller (karakter2, karakter3) — tilføjes manuelt i `CHARACTERS`-arrayet når behovet opstår
