# SPEC HARDWARE — Matrice de routage CV (BOM de référence)

> Compagnon de [SPEC_MATRICE_ROUTAGE_CV.md](SPEC_MATRICE_ROUTAGE_CV.md). Specs vérifiées juin 2026 sur datasheets constructeurs.
> **Prix/dispo = approximatifs** (les distributeurs n'exposent pas de tarif live) → à reconfirmer sur Digikey/Mouser avant achat.
> Niveau de confiance noté par ligne. Les valeurs de résistances des étages d'interface sont à calculer/simuler pour les rails choisis.

## 0. Le carrefour central : rails étroits I²C-natif vs rails larges parallèle

C'est **la** décision qui structure toute la carte :

| Axe | **ADG2188 / ADG2128** (I²C-natif) | **MT8816** (parallèle) |
|---|---|---|
| Commande | **I²C direct** ✔ (cohérent avec « tout I²C ») | Parallèle → **expandeur** (MCP23017 / PCF8575) requis |
| Rails / plage signal | **±5 V** (ou +12 V single) → **ne passe PAS du ±10 V sans atténuation** | VDD +12 / VEE −5 → **~12 Vpp bipolaire** direct |
| RON | 30 Ω (2188) / 50 Ω (2128) | ~65 Ω max |
| Prix | ~10–18 $/u | **~3–7 $/u** (moins cher) |
| Taille | 8×8 (64) / 8×12 (96) | 8×16 (128) |

**Conséquence de conception** : si on reste fidèle au « tout I²C » (ADG2188), **le monde interne de la matrice tourne à ±5 V**, et on **scale aux frontières** vers le ±10 V du modulaire (atténuer en entrée, amplifier en sortie). C'est propre et ça garde l'I²C partout.

➡️ **Reco** : **ADG2188 + échelle interne ±5 V + étages de scaling en bordure** pour le prototype (fidèle à l'archi I²C). Garder MT8816 en plan B si on veut du ±10 V natif bon marché en acceptant l'expandeur GPIO. **À trancher (cf. §8.4 de la spec).**

---

## 1. Crosspoint — routage

### ADG2188 (Analog Devices) — 8×8 I²C
| Spec | Valeur | Conf. |
|---|---|---|
| I²C | oui (jusqu'à 3,4 MHz), `LDSW` maj simultanée | high |
| Adresses/bus | A0/A1/A2 → **8 adresses** | high |
| Alim | **±5 V** dual ou **+12 V** single (span max 15 V) | high |
| Plage signal | rail-à-rail VSS→VDD | high |
| RON | ~30 Ω typ / 35 Ω max | high |
| Prix/dispo | ~10–18 $/u, actif, LFCSP-40 | med |

**Gotcha** : 30 Ω en série → **toujours bufferiser chaque sortie de colonne** vers une entrée haute-Z. Pour le 1V/oct, la variation de RON peut détoner → buffer + précision en aval.
Sources : [datasheet ADG2188](https://www.analog.com/media/en/technical-documentation/data-sheets/adg2188.pdf), [page ADI](https://www.analog.com/en/products/adg2188.html)

### ADG2128 (Analog Devices) — 8×12 I²C
| Spec | Valeur | Conf. |
|---|---|---|
| Adresses/bus | `1110`+A0/A1/A2 → **8 adresses** | high |
| Alim | ±5 V dual / +12 V single | high |
| RON | 50 Ω max | high |
| Canaux | 8×12 = 96 | high |

Plus dense (96 croisements) mais RON plus haut et asymétrie 8×12 à faire coller à la topologie. Source : [datasheet ADG2128](https://www.analog.com/media/en/technical-documentation/data-sheets/ADG2128.pdf)

### MT8816 (Microchip/CML) — 8×16 parallèle (plan B)
| Spec | Valeur | Conf. |
|---|---|---|
| Commande | **parallèle** (AX0-2, AY0-3, DATA, STROBE, RESET) — pas d'I²C | high |
| Alim | 4,5–13,2 V ; **VEE négatif → bipolaire** (~12 Vpp) | high |
| RON | ~65 Ω max @12 V | high |
| Prix | **~3–7 $/u** | med |

**Gotcha** : nécessite ~12 GPIO ou un expandeur (MCP23017/PCF8575). **Vérifier le cycle de vie du boîtier** (un variant PDIP AE1 signalé obsolète chez Digikey). Source : [datasheet MT8816](https://www.mouser.com/datasheet/2/268/Microchip_06182024_MT8816AE1-3459976.pdf)

---

## 2. Sources — DAC

### MCP4728 (Microchip) — quad 12 bit I²C + EEPROM
| Spec | Valeur | Conf. |
|---|---|---|
| Canaux / résolution | 4 × 12 bit, sortie tension RR | high |
| Vref interne | **2,048 V** (gain×2 → 0–4,096 V) | high |
| Sortie | **0–4,096 V unipolaire** (4,096 V seulement si VDD ≥ 5 V) | high |
| Alim | 2,7–5,5 V | high |
| Adresses/bus | défaut partagé **0x60**, reprogrammables en EEPROM (1 par 1) | high |
| EEPROM | oui (repart au dernier état) | high |
| Prix | ~2–4 $/u, MSOP-10 | med |

**Gotcha** : unipolaire 0–4,096 V → **étage offset+gain obligatoire** pour du CV bipolaire (§5A). Adresse par défaut partagée → **TCA9548A** ou ré-adressage EEPROM séquentiel.
**Résolution pitch** : 4,096 V / 4096 = 1 mV/LSB ≈ **1,2 cent/LSB** en 1V/oct → OK modulation/LFO/env, **limite pour du pitch multi-octave précis** (envisager 16 bit type AD5696 ou calibration logicielle pour les voies de hauteur). Sources : [datasheet MCP4728](https://ww1.microchip.com/downloads/en/devicedoc/22187e.pdf), [guide Adafruit](https://learn.adafruit.com/adafruit-mcp4728-i2c-quad-dac)

---

## 3. Master par bus de destination (option 3)

Deux écoles, à choisir :

### AD5254 (Analog Devices) — quad digipot I²C (simple, set-and-hold)
| Spec | Valeur | Conf. |
|---|---|---|
| Taps / canaux | 256 positions × 4, EEPROM | high |
| Adresses | AD0/AD1 → **4 adresses** (4 puces = 16 pots/bus) | high |
| Alim | +2,7…5,5 V (ou ±2,25…2,75 V) | high |
| Valeurs R | 1/10/50/100 kΩ | high |

**Gotcha clé** : domaine **≤ 5,5 V** → **ne passe pas du ±12 V en direct**. À utiliser comme **élément de gain dans le contre-réaction** d'un op-amp ±12 V où le **nœud du wiper reste dans ses rails** (trim/gain), pas comme atténuateur plein-signal. Source : [datasheet AD5253/5254](https://www.analog.com/media/en/technical-documentation/data-sheets/AD5253_5254.pdf)

### SSI2164 / V2164 — quad VCA (riche, piloté en tension par DAC)
| Spec | Valeur | Conf. |
|---|---|---|
| Canaux | 4 VCA indépendants | high |
| Contrôle | **tension** (expo, −33 mV/dB), chemin signal **en courant** | high |
| Alim | **±4…±18 V** (±12 V OK) | high |
| Gain | +20 dB → −100 dB | high |
| Prix | ~3–5 $ (SSI2164), V2164 moins cher | med |

**Bon candidat master par bus** piloté par une voie DAC (via op-amp de mise à l'échelle vers la fenêtre de contrôle −0,66…+3,3 V). **Gotcha** : réponse **exponentielle** (pas linéaire) et **I/O en courant** → op-amps I→V autour de chaque VCA (compte de pièces ↑). Sources : [datasheet SSI2164](https://www.soundsemiconductor.com/downloads/ssi2164datasheet.pdf)

> **Reco** : démarrer **digipot AD5254** (simple, I²C-direct, tenu) pour le master de bus ; réserver **SSI2164** si on veut un master VCA musical / contrôlable en tension plus tard.

---

## 4. Bus & op-amps

### TCA9548A / PCA9548A (TI/NXP) — mux I²C 8 canaux
Résout les collisions d'adresses (ex. plusieurs MCP4728 à 0x60). Adresses **0x70–0x77** (8 mux = 64 sous-bus). Ne passe **aucun analogique** — uniquement le bus I²C. ~1–2 $.
*(Un résultat de recherche l'a dit « obsolète » → **probablement faux/peu fiable**, reconfirmer sur ti.com.)* Source : [datasheet TCA9548A](https://www.ti.com/lit/ds/symlink/tca9548a.pdf)

### Op-amps (rails ±12 V)
| Part | Type | Rôle | Conf. |
|---|---|---|---|
| **TL074** | quad JFET | **workhorse** : buffers, sommation, mults (entrées haute-Z, pas cher) | high |
| **OPA4172** | quad précision | **voies 1V/oct** : offset ±0,2 mV, dérive ±0,3 µV/°C (préserve l'accord) | high |
| **NE5532** | dual audio | sommation audio bas bruit (mais bipolaire, biais d'entrée ↑, dual) | high |

**Reco** : **TL074** partout, **OPA4172** spécifiquement sur les chemins de **hauteur** (l'offset/dérive sur une ligne de pitch = désaccord). Sources : [OPA4172](https://www.ti.com/product/OPA4172), [TL074](https://www.st.com/resource/en/datasheet/tl074.pdf)

---

## 5. Étages d'interface analogique

### A. DAC 0–4,096 V → CV bipolaire (offset + gain)
Ampli **différentiel / sommateur inverseur** : `Vout = G·(Vdac − Vmid)`.
- **±5 V** : G ≈ 10/4,096 ≈ **2,44**, offset annulant le mi-échelle (~2,048 V).
- **±10 V** : G ≈ 20/4,096 ≈ **4,88**.
- Offset depuis une **référence de tension stable** (la stabilité de l'offset fixe la précision du 0 V). **OPA4172 + résistances 0,1 %** sur les voies de pitch. (Topologie = sortie type Mutable Instruments.)

### B. Protection des entrées
**Vers la matrice (ADG2188 ±5 V / MT8816)** : résistance série ~1 kΩ + **diodes de clamp vers les rails** (Schottky/BAV199/BAV99). ⚠️ **Un ADG2188 en ±5 V ne peut PAS passer du ±10 V** sans atténuation préalable → soit on atténue à ±5 V, soit MT8816 à rails plus larges.

**Vers une prise ADC ESP32 (0–3,3 V, jamais dépassé)** : diviseur + offset (ampli diff) **bufferisé** (l'ADC veut une source ≲ qqs kΩ) + **clamp dur en pin** (série 1–10 kΩ + Schottky vers 3,3 V et GND, ou BAT54S / TVS 3,3 V). L'ADC C3/S3 est non-linéaire près des rails et bruité ~12 bit → **bon pour du monitoring/retour, pas pour du pitch précis**.

---

## 6. Alimentation (sujet à part entière)

- **±12 V** (op-amps, switches, VCA) + **+5 V** (MCP4728 pour atteindre 4,096 V) + **3,3 V** (ESP).
- Hors châssis Eurorack : alim bipolaire à prévoir (brique ±12 externe, ou 5 V USB → DC-DC/charge-pump vers ±12 — **mais bruit de découpage sur le CV** → LDO propres + filtrage).
- **Masses** : séparer analogique/numérique, jointes en **un seul point (star ground)**. Déterminant pour un CV propre.

---

## 7. Stack de démarrage proposé (maquette 8×8, 1 tuile de chaque)

| Rôle | Pièce | Qté | Note |
|---|---|---|---|
| Crosspoint | **ADG2188** | 1 | 8×8, I²C |
| Sources | **MCP4728** | 1 | 4 CV → +4 voies via tuiles ajoutées |
| Master bus | **AD5254** | 1 | 4 masters de bus |
| Buffers/sommation | **TL074** | 2–3 | + **OPA4172** ×1 si voie pitch |
| Mux bus | **TCA9548A** | 1 | si > qqs tuiles identiques |
| Réf. tension | réf 2,048/4,096 V | 1 | offset stable |
| Protection | Schottky/BAT54S, R série | — | par I/O exposée |
| Alim | ±12 V + 5 V + 3,3 V | 1 | star ground |

Objectif maquette : valider **le bout-en-bout** (capteur → FluxRegistry → DAC → crosspoint → sommateur → sortie module, **et** retour ADC → MIDI/OSC) sur **un** 8×8 avant de paver la grille.

---

## 7bis. Estimation de prix (maquette 8×8)

> Prix unitaires **indicatifs** (achat à l'unité Mouser/Digikey/TME, **HT, hors port/TVA**, hors PCB), juin 2026. À l'unité = le plus cher. **Reconfirmer panier réel** (port mini distributeur ≈ 12–20 € sur une commande unique).

### Cœur — circuits intégrés

| Pièce | Qté | PU ~ | Sous-total |
|---|---|---|---|
| ADG2188 (crosspoint 8×8 I²C) | 1 | 10–18 € | ~14 € |
| MCP4728 (4× DAC) | 1 | 2–4 € | ~3 € |
| AD5254 (4× digipot, masters) | 1 | 3–6 € | ~4,5 € |
| TL074 (buffers/sommation) | 3 | ~1 € | ~3 € |
| OPA4172 (voie pitch précise) | 1 | 3–5 € | ~4 € |
| TCA9548A (mux I²C) | 1 | 1–2 € | ~1,5 € |
| Réf. tension (REF3040 / ADR4540) | 1 | 2–4 € | ~3 € |
| Protection (BAT54S, diodes, R série) | — | — | ~3 € |
| **Cœur CI** | | | **≈ 36 € (28–50 €)** |

### Total maquette réaliste

| Poste | ~ | Note |
|---|---|---|
| Cœur CI (ci-dessus) | 36 € | |
| **Alimentation ±12 / +5 / 3,3 V** | 15–40 € | **le plus gros variable** : brique ±12 ou DC-DC propre |
| Connectique | 5–25 € | 5 € si headers de banc ; 16× jacks 3,5 mm ≈ 20 € si « Eurorack » |
| Passifs (R 0,1 % sommation/scaling, caps) | ~10 € | |
| Protoboard / PCB proto | 5–15 € | |
| ESP32 XIAO C3/S3 | 0–7 € | 0 si déjà en main |
| **Total maquette** | | **≈ 80–130 €** |

### Synthèse
- **Si alim + ESP + connectique de banc déjà en main** → **~40–55 €** (CI + passifs).
- **Maquette complète from scratch** → **~100 €** (alim bipolaire propre + jacks = postes les plus lourds).
- Pour **une** tuile 8×8. Passage **16×16** ≈ ×4 sur crosspoints + sommateurs (**+50–70 €** de CI), pas sur le reste (bus/alim/ESP mutualisés).

---

## 8. Caveats honnêtes (à garder en tête)

- **Tous les crosspoints ont un RON 30–65 Ω** → bufferiser systématiquement, ne pas charger.
- **ADG2188/2128 en ±5 V ≠ ±10 V Eurorack** → atténuer aux frontières, ou MT8816 rails larges (perd l'I²C natif). **C'est l'arbitrage central.**
- **MCP4728** unipolaire + adresse par défaut partagée → offset+gain + TCA9548A.
- **AD5254** = élément de contrôle 5 V, pas un atténuateur plein-signal ±12 V.
- **SSI2164** = excellent master VCA mais **expo + courant** (op-amps autour).
- **Prix/dispo approximatifs** → reconfirmer distributeur ; **vérifier cycle de vie MT8816** et **statut TCA9548A** avant de figer la BOM.
