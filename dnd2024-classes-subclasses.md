# D&D 2024 PHB — Classes & Subclasses

12 klasser · 48 subclasses · Subclass vælges ved **level 3** medmindre andet er angivet.

---

## Barbarian
**Subclass label:** Path of the...
**Subclass level:** 3

- Path of the Berserker
- Path of the Wild Heart *(tidligere: Totem Warrior)*
- Path of the World Tree *(ny i 2024)*
- Path of the Zealot

---

## Bard
**Subclass label:** College of...
**Subclass level:** 3

- College of Dance *(ny i 2024)*
- College of Glamour
- College of Lore
- College of Valor

---

## Cleric
**Subclass label:** Domain
**Subclass level:** 1 *(vælges ved character creation)*

- Life Domain
- Light Domain
- Trickery Domain
- War Domain

---

## Druid
**Subclass label:** Circle of the...
**Subclass level:** 3

- Circle of the Land
- Circle of the Moon
- Circle of the Sea *(ny i 2024)*
- Circle of the Stars

---

## Fighter
**Subclass label:** Martial Archetype
**Subclass level:** 3

- Battle Master
- Champion
- Eldritch Knight
- Psi Warrior

---

## Monk
**Subclass label:** Warrior of...
**Subclass level:** 3

- Warrior of Mercy *(tidligere: Way of Mercy)*
- Warrior of Shadow *(tidligere: Way of Shadow)*
- Warrior of the Elements *(tidligere: Way of the Four Elements)*
- Warrior of the Open Hand *(tidligere: Way of the Open Hand)*

---

## Paladin
**Subclass label:** Oath of...
**Subclass level:** 3

- Oath of Devotion
- Oath of Glory
- Oath of the Ancients
- Oath of Vengeance

---

## Ranger
**Subclass label:** Ranger Conclave
**Subclass level:** 3

- Beast Master
- Fey Wanderer
- Gloom Stalker
- Hunter

---

## Rogue
**Subclass label:** Roguish Archetype
**Subclass level:** 3

- Arcane Trickster
- Assassin
- Soulknife
- Thief

---

## Sorcerer
**Subclass label:** Sorcerous Origin
**Subclass level:** 3

- Aberrant Sorcery *(tidligere: Aberrant Mind)*
- Clockwork Sorcery *(tidligere: Clockwork Soul)*
- Draconic Sorcery *(tidligere: Draconic Bloodline)*
- Wild Magic Sorcery *(tidligere: Wild Magic)*

---

## Warlock
**Subclass label:** Otherworldly Patron
**Subclass level:** 3

- Archfey Patron
- Celestial Patron
- Fiend Patron
- Great Old One Patron

---

## Wizard
**Subclass label:** Arcane Tradition
**Subclass level:** 3

- Abjurer *(tidligere: School of Abjuration)*
- Diviner *(tidligere: School of Divination)*
- Evoker *(tidligere: School of Evocation)*
- Illusionist *(tidligere: School of Illusion)*

---

## Implementation notes til Claude Code

```js
const CLASS_SUBCLASSES = {
  Barbarian: { label: "Path of the", level: 3, subclasses: ["Path of the Berserker", "Path of the Wild Heart", "Path of the World Tree", "Path of the Zealot"] },
  Bard:      { label: "College of",  level: 3, subclasses: ["College of Dance", "College of Glamour", "College of Lore", "College of Valor"] },
  Cleric:    { label: "Domain",      level: 1, subclasses: ["Life Domain", "Light Domain", "Trickery Domain", "War Domain"] },
  Druid:     { label: "Circle of",   level: 3, subclasses: ["Circle of the Land", "Circle of the Moon", "Circle of the Sea", "Circle of the Stars"] },
  Fighter:   { label: "Archetype",   level: 3, subclasses: ["Battle Master", "Champion", "Eldritch Knight", "Psi Warrior"] },
  Monk:      { label: "Warrior of",  level: 3, subclasses: ["Warrior of Mercy", "Warrior of Shadow", "Warrior of the Elements", "Warrior of the Open Hand"] },
  Paladin:   { label: "Oath of",     level: 3, subclasses: ["Oath of Devotion", "Oath of Glory", "Oath of the Ancients", "Oath of Vengeance"] },
  Ranger:    { label: "Conclave",    level: 3, subclasses: ["Beast Master", "Fey Wanderer", "Gloom Stalker", "Hunter"] },
  Rogue:     { label: "Archetype",   level: 3, subclasses: ["Arcane Trickster", "Assassin", "Soulknife", "Thief"] },
  Sorcerer:  { label: "Origin",      level: 3, subclasses: ["Aberrant Sorcery", "Clockwork Sorcery", "Draconic Sorcery", "Wild Magic Sorcery"] },
  Warlock:   { label: "Patron",      level: 3, subclasses: ["Archfey Patron", "Celestial Patron", "Fiend Patron", "Great Old One Patron"] },
  Wizard:    { label: "Tradition",   level: 3, subclasses: ["Abjurer", "Diviner", "Evoker", "Illusionist"] },
};
```

### Dropdown-logik
- Populer subclass-dropdown dynamisk når class ændres
- Vis `"— Vælges ved level X —"` som disabled option hvis `C.level < subclassLevel`
- Nulstil subclass-valg ved class-skift
- Cleric er eneste klasse der vælger subclass ved level 1 (character creation)
