# whiteNoiseGenerator

> Générateur de **bruit blanc** à transistors, amplifié en deux étages — source de
> modulation/audio, conçu pour fonctionner en **alim simple** avec une masse virtuelle.

| | |
|---|---|
| **Famille** | source de signal |
| **Tier** | 1 |
| **Outil / source de vérité** | Fritzing `fritzing/whiteNoiseGenerator.fzz` ⟵ ⚠️ à committer |
| **Statut** | ✅ esquisse (schéma) |
| **Alim** | Vcc simple (≥ ~12 V conseillé pour l'avalanche) + **Vref** (= Vcc/2) |

## Fonction

La jonction **base-émetteur de Q1 (2N3904) polarisée en inverse** sert de source de bruit
d'avalanche (via R1 150 kΩ depuis Vcc) ; **Q2** amplifie. Le bruit est **couplé en
alternatif** par C1 0,47 µF vers un 1ᵉʳ étage (section A du TL082, polarisé sur **Vref**),
puis C2 0,47 µF → R4 12 kΩ vers un 2ᵉ étage inverseur (section B, contre-réaction R5 47 kΩ
→ **gain ≈ 47/12 ≈ 3,9**). Sortie **T1 (Analog Out)**.

> **Dépendance** : la polarisation utilise **Vref = Vcc/2** fournie par le module
> [virtualGround](../virtualGround/).

## Entrées / sorties

| Repère | Type | Description |
|---|---|---|
| Vcc | alim | rail positif (≥ ~12 V pour une avalanche fiable) |
| GND | alim | 0 V |
| Vref | entrée | Vcc/2 (polarisation des étages) |
| T1 / Analog Out | sortie | bruit blanc amplifié |

## BOM (indicative)

| Réf | Valeur | Note |
|---|---|---|
| Q1, Q2 | 2N3904 | Q1 = source de bruit (B-E en inverse), Q2 = ampli |
| U1 | TL082 | ampli op double (2 étages) |
| R1 | 150 kΩ | polarisation Q1 |
| R2, R3 | 1 MΩ | référence Vref vers entrées + |
| R4 | 12 kΩ | entrée du 2ᵉ étage |
| R5 | 47 kΩ | contre-réaction (gain ≈ 3,9) |
| C1, C2 | 0,47 µF | couplage AC |

## Netlist (reconstruction Fritzing)

Alim simple : `VCC` · `GND` · `VREF` (= sortie du [virtualGround](../virtualGround/)).
Nets : `N_NOISE`(1) · `N_BASE`(2) · `N_A_IN`(3) · `N_A_OUT`(4) · `N_C2`(5) · `N_B_IN`(7) · `N_OUT`(8).

**Brochage 2N3904 (TO-92)** : 1 = Émetteur · 2 = Base · 3 = Collecteur.

| Composant | Broche | Net |
|---|---|---|
| R1 150 kΩ | a / b | VCC / N_NOISE |
| Q1 2N3904 | 1 (E) | N_NOISE |
| Q1 2N3904 | 2 (B) | N_BASE |
| Q1 2N3904 | 3 (C) | **NC** (collecteur en l'air) |
| Q2 2N3904 | 3 (C) | N_NOISE |
| Q2 2N3904 | 2 (B) | N_BASE |
| Q2 2N3904 | 1 (E) | GND |
| C1 0,47 µF | a / b | N_NOISE / N_A_IN |
| R2 1 MΩ | a / b | VREF / N_A_IN |
| U1-A TL082 | 3 (IN+) | N_A_IN |
| U1-A TL082 | 2 (IN−) | N_A_OUT *(suiveur)* |
| U1-A TL082 | 1 (OUT) | N_A_OUT |
| C2 0,47 µF | a / b | N_A_OUT / N_C2 |
| R4 12 kΩ | a / b | N_C2 / N_B_IN |
| R3 1 MΩ | a / b | VREF / pin 5 |
| U1-B TL082 | 5 (IN+) | VREF *(via R3)* |
| U1-B TL082 | 6 (IN−) | N_B_IN |
| R5 47 kΩ | a / b | N_B_IN / N_OUT *(contre-réaction)* |
| U1-B TL082 | 7 (OUT) | N_OUT |
| U1 TL082 | 8 (V+) / 4 (V−) | VCC / GND |
| Jack T1 | — | N_OUT |

→ **Cœur de bruit** : Q1 = jonction B-E en inverse (avalanche), collecteur en l'air ;
Q2 = ampli émetteur commun (charge R1, sortie collecteur = N_NOISE), couplé par C1.
→ **Étage A** = suiveur (buffer) ; **étage B** = ampli inverseur, **gain = R5/R4 ≈ 3,9**.
Entrées + polarisées sur VREF (alim simple).

## Fichiers

- `fritzing/` — ⚠️ `.fzz` source manquant (à reconstruire/committer)
- `export/` — `schema_whiteNoiseGen.jpg`, `schema_whiteNoiseGenerator.svg`
- `refs/` — `2N3904.jpg`, `tl082.jpg` (brochages)

## À faire / questions ouvertes

- [ ] Confirmer la tension Vcc mini pour une avalanche fiable du 2N3904 (~9–12 V).
- [ ] Committer le fichier source `.fzz`.
- [ ] Caractériser le spectre / niveau de sortie (blanc vs coloré) et ajuster le gain.
- [ ] Option : filtre pour dériver un **bruit rose**.
