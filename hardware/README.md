# hardware/

Conception électronique de la matrice CV. Projet KiCad à venir.

## Cible maquette

Une **tuile 8×8** bout-en-bout (cf. [`../docs/SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md`](../docs/SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md) §7) :

- 1× ADG2188 (crosspoint 8×8 I²C)
- 1× MCP4728 (4 sources DAC) + étages offset/gain → CV bipolaire
- 1× AD5254 (4 masters de bus, digipot)
- TL074 (buffers/sommation) + 1× OPA4172 (voie pitch)
- 1× TCA9548A (mux I²C)
- Référence de tension, protections d'entrée, alim ±12 / +5 / 3,3 V (star ground)

## Décisions hardware à trancher

- **Rails** : ADG2188 ±5 V (I²C-natif, *reco*) + scaling aux frontières ±10 V, vs MT8816 ±12/−5 (bipolaire natif, mais expandeur GPIO).
- **Master par bus** : digipot AD5254 (simple) vs VCA SSI2164 (musical, piloté en tension).
- **Topologie d'alim** : ±12 externe (type Eurorack) vs génération embarquée.

## Sous-dossiers

- `bom/` — nomenclatures (la BOM de référence détaillée est dans `docs/`).
- *(à venir)* projet KiCad : schéma, PCB, empreintes.
