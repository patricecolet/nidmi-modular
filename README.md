# nidmi-modular

Interface de contrôle d'un **synthé modulaire** par NiDMI : une **matrice de routage CV** hybride, extensible, commandée en I²C.

`nidmi-modular` ponte trois mondes en un seul hub de signaux :

```
   CV modulaire  ⇄  numérique NiDMI (FluxRegistry / MappingEngine)  ⇄  réseau MIDI / OSC
```

Toute la matrice est commandée sur **un seul bus I²C** — on ajoute de la capacité CV en **empilant des tuiles**, sans consommer les broches de l'ESP32.

## Statut

**Conception** (specs figées, hardware/firmware non implémentés). Issu d'une parenthèse de design sur la branche `feat/analog-modules` de [`NiDMI`](../NiDMI).

## Place dans l'écosystème

- [`nidmi-core`](../nidmi-core) — couche commune MIDI/réseau/I²C. `nidmi-modular` s'appuie dessus (cible).
- [`NiDMI`](../NiDMI) — application capteurs/actuateurs → MIDI temps réel. `nidmi-modular` réutilise ses concepts `FluxRegistry` / `MappingEngine` comme cerveau du patch.

## Concept en bref

- **Matrice M→N partagée**, non-bloquante, avec **sommation** (bus à masse virtuelle).
- **Niveaux** : par source en numérique (DAC) + un master par bus (digipot).
- **Trois tuiles I²C** : source (MCP4728 DAC), crosspoint (ADG2188), destination (sommateur + digipot).
- **Cerveau** : un singleton `RoutingFabric` piloté par `FluxRegistry` / `MappingEngine`, patch persisté en NVS, UI patchbay web.

## Arborescence

```
docs/       Specs de conception
  SPEC_MATRICE_ROUTAGE_CV.md            ← architecture & modèle logiciel
  SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md   ← BOM de référence + estimation prix
hardware/   Conception électronique (KiCad), BOM
  bom/
firmware/   Firmware de contrôle (RoutingFabric), cible nidmi-core
```

## Par où commencer

1. Lire [`docs/SPEC_MATRICE_ROUTAGE_CV.md`](docs/SPEC_MATRICE_ROUTAGE_CV.md) (concept, décisions actées, questions ouvertes).
2. Lire [`docs/SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md`](docs/SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md) (pièces, prix, étages d'interface).
3. Décisions à trancher : **rails** (±5 V I²C ADG2188 *(reco)* vs ±12/−5 MT8816) et **pilotage du patch** (UI / scripts / réseau).
4. Maquette : une tuile **8×8** bout-en-bout avant de paver la grille.

## Contribuer

Installation de l'écosystème et workflow de branches : [`CONTRIBUTING.md`](CONTRIBUTING.md).
