# SPEC — Matrice de routage CV (fabric hybride NiDMI ↔ modulaire ↔ MIDI/OSC)

> Statut : **note de conception** (concept figé, non implémenté). Parenthèse de design sur la branche `feat/analog-modules`.
> Évaluation matérielle détaillée : voir [SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md](SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md).

## 1. Objectif

Faire de NiDMI l'**interface de contrôle d'un synthé modulaire** via une **matrice de routage CV** extensible. La matrice est un *hub de signaux hybride* qui ponte trois mondes simultanément :

```
   CV modulaire  ⇄  numérique NiDMI (FluxRegistry / MappingEngine)  ⇄  réseau MIDI / OSC
```

Toute la matrice est commandée par NiDMI sur **un seul bus I²C** — on ajoute de la capacité CV en **empilant des tuiles**, pas en consommant des broches de l'ESP32.

## 2. Décisions actées (parenthèse de design)

| # | Décision | Choix retenu |
|---|----------|--------------|
| 1 | Rôle de NiDMI | Cerveau **et** pont (génère du CV **et** traduit gestes/réseau en CV) |
| 2 | Topologie | **Matrice M→N partagée** (fabric commune, pas une matrice par composant) |
| 3 | Type de signaux | **CV / contrôle lent uniquement** (pas d'audio dans la matrice scannée) |
| 4 | Mélange | **Oui** — sommation de plusieurs sources sur une destination |
| 5 | Niveaux de mix | **Option (3)** : sommation à gain unité + **niveau par source (numérique, DAC)** + **un maître réglable par bus de destination (digipot)** |
| 6 | Transport de commande | **I²C**, tuiles empilables, découverte au boot |

> Non tranché (questions ouvertes, §8) : **qui pilote le patch en priorité** (UI manuelle / scripts vivants / rappel réseau).

## 3. Architecture matérielle

### 3.1 Topologie : grille non-bloquante + sommation

- **Non-bloquant** : toutes les sources sur des rails communs, toutes les destinations sur des rails communs ; la grille M×N est *pavée* de crosspoints (ex. fabric 16×16 = bloc 2×2 de crosspoints 8×8 partageant lignes/colonnes). « Ajouter une tuile » = étendre la grille d'un bloc de lignes ou de colonnes.
- **Sommation à masse virtuelle** : chaque destination est un **ampli sommateur** (entrée en masse virtuelle). Chaque crosspoint **injecte une source via une résistance** dans ce nœud. Plusieurs sources fermées sur une même destination **s'additionnent** (au lieu de se court-circuiter). C'est ce qui *lève* l'ancien invariant « une source par destination ».

### 3.2 Taxonomie des tuiles (toutes I²C)

| Tuile | Puce pivot | Rôle | Verbe NiDMI (I²C) |
|---|---|---|---|
| **Source** | MCP4728 (4× DAC 12 bit) | génère / buffer les CV | **niveau par source** = valeur DAC |
| **Crosspoint** | ADG2188 (8×8) / ADG2128 (8×12) | routage tenu, on/off | **qui rejoint quel bus** |
| **Destination** | sommateur + digipot I²C | mixe + master + sortie | **master par bus** = digipot ; sortie → module *ou* prise ADC |

- **Extension d'adresses** : chaque puce a 2–3 broches d'adresse → quelques tuiles identiques par bus. Au-delà, un **mux I²C TCA9548A** (8 sous-bus) réutilise les mêmes adresses → empilement quasi illimité.
- **Le pont numérique** se fait à deux endroits :
  - une **source** peut être pilotée par du **MIDI/OSC entrant** (réseau → valeur → DAC) ;
  - une **destination** peut être une **prise ADC de NiDMI** (bus sommé → resamplé → ressorti en **MIDI/OSC**).

## 4. Les quatre types de « bouts »

- **Sources** : signaux des modules (CV externe) **+** signaux ESP (DAC pilotés par capteurs, LFO internes, ou MIDI/OSC entrant).
- **Destinations** : entrées de modules analo **+** prises ADC NiDMI (retour → MIDI/OSC).

N'importe quel nœud peut être tapé par NiDMI et renvoyé sur le réseau ; n'importe quel message réseau peut devenir une source CV.

## 5. Architecture logicielle

Nouveau singleton **`RoutingFabric`** (à côté de `g_componentManager` / `g_midiRouter` dans `Globals.h`).

### 5.1 Modèle de données

```
Source      = { nom, phys:(tuile DAC, canal) | entrée externe, valeur, niveau }
Destination = { nom, phys:(tuile somme, canal digipot), sortie: MODULE | ADC_TAP,
                master, sources_connectées: Set<Source> }
Crosspoint  = (source, destination) → (adresse ADG2188, x, y)
Patch       = { croisements fermés } + { niveaux sources } + { masters bus }   → persisté NVS
```

### 5.2 Verbes

- `connect(source, dest)` / `disconnect(source, dest)` → écriture crosspoint
- `setSourceLevel(source, v)` → écriture DAC (MCP4728)
- `setBusMaster(dest, g)` → écriture digipot
- `scan()` → découverte des tuiles I²C au boot (énumère les pools source/destination)
- `savePatch()` / `recallPatch(slot)` → NVS (snapshots, très « modulaire »)

### 5.3 Liens existants réutilisés

- **`FluxRegistry`** : les valeurs nommées produites par les `process()` des composants deviennent des **valeurs de source** poussées vers les DAC.
- **`MappingEngine`** : peut décider *quel croisement fermer* et *quelle valeur sort chaque DAC* → patch « vivant » piloté par le signal.
- **`I2CManager`** : porte le bus.
- **NVS** : persistance des patchs (NiDMI sait déjà le faire).
- **UI** : nouveau **widget grille de patchbay** (sources × destinations), clic = connexion ; une colonne peut avoir plusieurs points (mélange), plus de notion de « conflit ». **C'est le principal morceau neuf côté front.**

## 6. Le niveau de mix (option 3, détail)

- **Par source** : numérique, quasi gratuit — l'amplitude est la valeur DAC. NiDMI « mixe » en bougeant les DAC.
- **Par bus** : un **digipot I²C** dans le gain du sommateur de chaque destination → un master par bus.
- **Pas de niveau par croisement** (le crosspoint est on/off) → on évite l'explosion de BOM d'une matrice type Synthi (digipot/VCA par point).

**Piège honnête** : si une source part sur **plusieurs bus** et qu'on veut un **niveau différent par bus**, le réglage par-source ne suffit pas (amplitude globale). Solutions : dédier une **2ᵉ voie DAC**, ou accepter une contribution égale partout. Vivable pour un instrument d'atelier.

## 7. Contraintes & métriques à surveiller

- **Bande passante I²C** : à 400 kHz–1 MHz, pousser quelques dizaines d'écritures DAC par tick de 10 ms (boucle 100 Hz) passe large pour du **contrôle lent**. *La* métrique à surveiller si on multiplie les sources internes rafraîchies vite → **le bus devient la limite, pas les broches.**
- **Buffering analogique** : un buffer par source, un sommateur par destination → `M + N` voies d'op-amps. C'est ce qui mange la surface de carte / la BOM, pas le digital.
- **Niveau de tension** : DAC sort 0–VDD, modulaire attend ±5/±10 V → **étage de level-shift / ampli** obligatoire. Voir doc hardware.
- **Alimentation bipolaire** : sources/sommateurs/crosspoints bipolaires → besoin d'une alim **±** (ex. ±12 V Eurorack) en plus du 3,3 V ESP. Sujet à part entière (doc hardware).
- **Hot-plug** : ajouter une tuile à chaud = re-scan I²C → logique à écrire.

## 8. Questions ouvertes

1. **Qui pilote le patch en priorité ?** UI patchbay manuelle (geste) / scripts `MappingEngine` (patch vivant) / réseau MIDI-OSC (CC-PC ferme un croisement ou rappelle un snapshot). Probablement les trois — la priorité oriente l'UI et le modèle de presets.
2. Niveau **indépendant par bus** pour une source fan-out : 2ᵉ voie DAC vs contribution égale (cf. §6).
3. Dimension cible de la première fabric (sweet spot proposé : **8×8 ou 16×8**).
4. Topologie d'alimentation (±12 V externe Eurorack vs génération embarquée).

## 9. Suite

- [x] Évaluation matérielle / BOM de référence → [SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md](SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md)
- [ ] **Trancher le carrefour des rails** (cf. doc hardware §0) : ADG2188 I²C-natif + échelle interne ±5 V + scaling en bordure *(reco)*, vs MT8816 ±12/−5 natif + expandeur GPIO
- [ ] Trancher §8.1 (pilotage du patch : UI manuelle / scripts vivants / rappel réseau)
- [ ] Maquette d'une tuile minimale (1 ADG2188 8×8 + 1 sommateur + 1 MCP4728 + 1 AD5254) pour valider le bout-en-bout
