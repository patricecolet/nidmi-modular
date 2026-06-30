# Contribuer

`nidmi-modular` fait partie d'un écosystème de **dépôts sœurs** (le code attend un layout
côte à côte, p. ex. `../nidmi-core`). On les gère avec [vcstool](https://github.com/dirk-thomas/vcstool).

## 1. Récupérer l'écosystème

```bash
pip install vcstool
mkdir nidmi-ws && cd nidmi-ws
git clone https://github.com/patricecolet/nidmi-modular.git
vcs import < nidmi-modular/nidmi.repos
```

Résultat (en sibling) :

```
nidmi-ws/
├── nidmi-core/          couche commune MIDI/réseau/I²C
├── NiDMI/               app capteurs → MIDI temps réel
├── nidmi-modular/       ce dépôt (matrice CV + modules)
└── nidmi-circuit-lab/   builds Fritzing
```

## 2. Travailler sur une branche

Une fonctionnalité touche en général **un seul** dépôt. Dedans, git classique :

```bash
git switch -c feat/ma-feature
# ... commits ...
git push -u origin feat/ma-feature
```

Puis ouvrir une **Pull Request** vers `main`.

## 3. Rester à jour

```bash
vcs pull                 # avance les `main` des 4 dépôts (non destructif)
vcs status               # état de tous les dépôts d'un coup
```

Pour remettre sa branche à niveau : `git fetch origin && git rebase origin/main`.

## 4. Fonctionnalité sur plusieurs dépôts

Si un changement touche aussi une dépendance (ex. `nidmi-core`) :

- **même nom de branche** dans chaque dépôt concerné ;
- chaque PR **référence l'autre** ;
- merger le dépôt dépendance d'abord.

## Outil — Fritzing

Les modules simples sont conçus dans **Fritzing**. Binaires prêts à l'emploi (gratuits) :
[releases de `nidmi-circuit-lab`](https://github.com/patricecolet/nidmi-circuit-lab/releases)
(macOS Apple Silicon, Windows x64).

- **macOS** (app non notarisée) : au 1er lancement, débloquer avec
  `xattr -dr com.apple.quarantine /Applications/Fritzing.app`.
- **Windows** (binaire non signé) : SmartScreen → *Informations complémentaires ▸ Exécuter quand même*.
- Sinon, build manuel : [`docs/BUILD_FRITZING.md`](docs/BUILD_FRITZING.md).

## Où contribuer

- **Modules à construire** : [`docs/ROADMAP.md`](docs/ROADMAP.md) (checklist en tête).
- **Convention par module** : [`docs/modules/README.md`](docs/modules/README.md)
  (`fritzing/` ou `kicad/` + `export/` + `refs/` + fiche `README.md` ; source de vérité
  = le `.fzz`/projet KiCad, jamais l'export).
- **Architecture & specs** : [`docs/SPEC_MATRICE_ROUTAGE_CV.md`](docs/SPEC_MATRICE_ROUTAGE_CV.md).
