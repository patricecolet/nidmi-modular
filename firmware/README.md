# firmware/

Firmware de contrôle de la matrice CV. Cible : s'appuyer sur [`nidmi-core`](../../nidmi-core).

## Cœur : le singleton `RoutingFabric`

Voir [`../docs/SPEC_MATRICE_ROUTAGE_CV.md`](../docs/SPEC_MATRICE_ROUTAGE_CV.md) §5 pour le modèle complet.

```
Source      = { nom, phys:(tuile DAC, canal) | entrée externe, valeur, niveau }
Destination = { nom, phys:(tuile somme, canal digipot), sortie: MODULE | ADC_TAP,
                master, sources_connectées: Set<Source> }
Crosspoint  = (source, destination) → (adresse ADG2188, x, y)
Patch       = croisements fermés + niveaux sources + masters bus   → persisté NVS
```

Verbes : `connect` / `disconnect` (crosspoint), `setSourceLevel` (DAC), `setBusMaster` (digipot), `scan()` (découverte I²C au boot), `savePatch` / `recallPatch(slot)` (NVS).

## Réutilisé depuis l'écosystème

- **`FluxRegistry` / `MappingEngine`** (concepts NiDMI) : cerveau du patch — décide quel croisement fermer et quelle valeur sort chaque DAC.
- **`I2CManager`** : porte le bus (+ TCA9548A pour l'extension d'adresses).
- **NVS** : persistance des patchs (snapshots).

## À trancher avant de coder

- **Qui pilote le patch en priorité** : UI patchbay web / scripts `MappingEngine` (patch vivant) / réseau MIDI-OSC (CC-PC ferme un croisement ou rappelle un snapshot).
- Forme du paquet : app autonome consommant `nidmi-core`, ou module greffé dans `NiDMI`.
