# virtualGround

> Masse virtuelle **Vref = Vcc/2** bufferisée — permet de traiter de l'audio/CV
> bipolaire sur une **alimentation simple**.

| | |
|---|---|
| **Famille** | infrastructure (référence d'alim) |
| **Tier** | 1 |
| **Outil / source de vérité** | Fritzing `fritzing/virtualGround.fzz` ⟵ ⚠️ à committer |
| **Statut** | ✅ esquisse (schéma + breadboard + stripboard) |
| **Alim** | Vcc simple **10–30 Vdc** |

## Fonction

Pont diviseur **R1/R2 = 100 kΩ / 100 kΩ** entre Vcc et GND → point milieu à **Vcc/2**,
**bufferisé** par un ampli op monté en suiveur (sortie rebouclée sur l'entrée −, entrée +
sur le milieu du pont). Un condensateur de filtrage sur le point milieu découple la
référence. Sortie **Vref = Vcc/2** à basse impédance, utilisée comme masse de signal par
les autres modules en alim simple (ex. [whiteNoiseGenerator](../whiteNoiseGenerator/)).

## Entrées / sorties

| Repère | Type | Description |
|---|---|---|
| Vcc | alim | +10…30 V |
| GND | alim | 0 V |
| Vref | sortie | Vcc/2 bufferisé (basse impédance) |

## BOM (indicative)

| Réf | Valeur | Note |
|---|---|---|
| U1 | TL071 (ou TL082) | ampli op en suiveur |
| R1, R2 | 100 kΩ | pont diviseur |
| C1 | 100 µF / 50 V | filtrage du milieu (variante : 4,7 µF) |

> **Variante retenue : TL071 + 100 µF.** (L'esquisse `virtualGroundSchematics.jpg`
> montre une variante historique TL082 + 4,7 µF, écartée.)

## Netlist (reconstruction Fritzing)

Variante canonique : **TL071 + 100 µF**, alim simple Vcc/GND.
Nets : `VCC` · `GND` · `VDIV` (milieu du pont) · `VREF` (sortie).

| Composant | Broche | Net |
|---|---|---|
| R1 100 kΩ | a / b | VCC / VDIV |
| R2 100 kΩ | a / b | VDIV / GND |
| C1 100 µF/50 V | + / − | VDIV / GND |
| U1 TL071 | 3 (IN+) | VDIV |
| U1 TL071 | 2 (IN−) | VREF *(contre-réaction)* |
| U1 TL071 | 6 (OUT) | VREF |
| U1 TL071 | 7 (V+) | VCC |
| U1 TL071 | 4 (V−) | GND |
| U1 TL071 | 1, 5, 8 | NC |
| Jack OUT | — | VREF |

→ Suiveur à gain unité : **VREF = Vcc/2** bufferisé. *(L'AOP est noté « U2 » sur l'esquisse → renuméroté U1.)*

## Fichiers

- `fritzing/` — ⚠️ `.fzz` source manquant (à reconstruire/committer)
- `export/` — `schema_virtualGround.svg/.png`, `virtualGroundSchematics.jpg` (variante)
- `refs/` — `tl071.jpg` (brochage)

## À faire / questions ouvertes

- [x] Ampli/filtrage tranchés : **TL071 + 100 µF**.
- [ ] Committer le fichier source `.fzz`.
- [ ] Vérifier le courant de sortie max (capacité de la masse virtuelle à encaisser la somme des bus).
