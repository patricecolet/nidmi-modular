# Builder Fritzing depuis les sources (gratuit) — macOS

Fritzing est open-source (GPL) : seul le **binaire** est payant. On le **compile** pour
l'avoir gratuitement et garder ses vues **breadboard / stripboard** (raison pour laquelle
on le préfère à KiCad pour les modules simples).

> 💡 **Binaires prêts à l'emploi** (macOS arm64, Windows x64) :
> [releases de `nidmi-circuit-lab`](https://github.com/patricecolet/nidmi-circuit-lab/releases).
> Ce document ne sert qu'au **build manuel** (autres plateformes, ou débogage de la CI).

> Cible : **macOS arm64** (Apple Silicon). Le build vit **hors** du dépôt `nidmi-modular`
> (ex. `~/src/fritzing/`), seul ce doc est versionné.

## Prérequis (déjà présents sur cette machine)

Homebrew · Xcode Command Line Tools · git · clang · boost.

## 1. Dépendances

```bash
brew install qt libgit2 ngspice quazip cmake
# boost : déjà installé (1.86)
```

- `qt` = Qt 6.x (requis ≥ 6.5.3).
- `libgit2`, `ngspice`, `quazip` : dépendances de Fritzing.

## 2. Espace de travail + sources

Fritzing attend ses dépendances *sœurs* dans le même dossier parent.

```bash
mkdir -p ~/src/fritzing && cd ~/src/fritzing
git clone https://github.com/fritzing/fritzing-app.git
git clone https://github.com/fritzing/fritzing-parts.git
# dépendances sœurs non couvertes par brew (si le build les réclame) :
git clone https://github.com/sebholt/svgpp.git        # ou le svgpp officiel
git clone https://github.com/AngusJohnson/Clipper2.git # Clipper
```

Arborescence attendue :

```
~/src/fritzing/
├── fritzing-app/
├── fritzing-parts/
├── svgpp/        (si réclamé)
└── Clipper.../   (si réclamé)
```

## 3. Build

```bash
cd ~/src/fritzing/fritzing-app
qmake phoenix.pro            # qmake fourni par brew qt (sinon: $(brew --prefix qt)/bin/qmake)
make -j$(sysctl -n hw.ncpu)
```

## 4. Premier lancement (génération de la base de pièces)

La base de pièces se génère **une fois** avec `-db`, en pointant `fritzing-parts` :

```bash
./Fritzing.app/Contents/MacOS/Fritzing \
  -db   "$HOME/src/fritzing/fritzing-parts/parts.db" \
  -f    "$HOME/src/fritzing/fritzing-app/" \
  -parts "$HOME/src/fritzing/fritzing-parts/"
```

Lancements suivants : ouvrir `Fritzing.app` normalement.

## Si le build casse sur macOS récent

L'upstream traîne parfois sur les dernières versions de macOS/Qt. Fork communautaire qui
cible explicitement « macOS build improvements + modern Qt6 » :

```bash
git clone https://github.com/bozza-man/fritzing-app.git
```

⚠️ **Code tiers** : à relire avant de compiler/exécuter (ce n'est pas l'upstream officiel).

## Notes

- `qmake introuvable` → ajouter Qt au PATH : `export PATH="$(brew --prefix qt)/bin:$PATH"`.
- Erreurs `libgit2`/version d'API → la version brew peut différer de celle attendue ;
  se rabattre sur une libgit2 0.28.x compilée en local (cf. wiki officiel) si besoin.
- Alternative gratuite si on renonce au breadboard : **KiCad** (les netlists des fiches
  modules s'y transcrivent telles quelles).

Sources : [Wiki officiel — Building Fritzing](https://github.com/fritzing/fritzing-app/wiki/1.-Building-Fritzing) ·
[Siytek — Build Fritzing from source](https://siytek.com/build-fritzing/) ·
[Fork macOS/Qt6](https://github.com/bozza-man/fritzing-app)
