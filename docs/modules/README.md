# Modules

Dossier par module de l'écosystème `nidmi-modular`. Énumération et ordre de
réalisation : voir [`../ROADMAP.md`](../ROADMAP.md).

## Convention

```
docs/modules/<nomModule>/
  README.md          fiche : rôle, I/O, alim, BOM, statut
  fritzing/<nom>.fzz  SOURCE DE VÉRITÉ (module simple) — à committer
  kicad/              SOURCE DE VÉRITÉ (module complexe) à la place de fritzing/
  export/            schematic.png · breadboard.png · stripboard.png (générés)
  refs/              datasheets, brochages
```

- **camelCase** pour les noms de dossier (cohérent avec l'existant).
- La **source de vérité** est le `.fzz` (Fritzing) ou le projet KiCad — jamais l'export seul.
- Copier [`_TEMPLATE/`](_TEMPLATE/) pour démarrer un nouveau module.
- La matrice CV (cœur complexe) vit dans [`../../hardware/`](../../hardware/), pas ici.

## Modules existants

| Module | Famille | Outil | Statut |
|---|---|---|---|
| [virtualGround](virtualGround/) | infrastructure | Fritzing | ✅ esquisse (⚠️ `.fzz` manquant) |
| [whiteNoiseGenerator](whiteNoiseGenerator/) | source | Fritzing | ✅ esquisse (⚠️ `.fzz` manquant) |

> ⚠️ **Sources `.fzz` manquantes** : seuls les exports (SVG/PNG/JPG) sont présents.
> Tant que les `.fzz` ne sont pas commités, ces modules ne sont pas réellement
> réeditables/versionnés (cf. [`../ROADMAP.md`](../ROADMAP.md)).
