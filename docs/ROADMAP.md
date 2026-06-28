# Feuille de route des modules

> Énumération des modules matériels de l'écosystème `nidmi-modular`.
> Voir [SPEC_MATRICE_ROUTAGE_CV.md](SPEC_MATRICE_ROUTAGE_CV.md) (architecture) et
> [SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md](SPEC_MATRICE_ROUTAGE_CV_HARDWARE.md) (BOM).

## Conventions

- **Outil** : `Fritzing` pour les modules simples (gain : vues breadboard + stripboard
  pour le proto/doc) · `KiCad` pour les modules denses / fine-pitch / multicouche.
- **Source de vérité** : le fichier `.fzz` (Fritzing) ou le projet KiCad — **commité**,
  jamais l'export SVG/PNG seul.
- **Rangement** : un dossier par module dans `docs/modules/<nomModule>/`
  (`fritzing/` ou `kicad/` + `export/` + `refs/` + `README.md`). La matrice reste
  dans `hardware/`.

**Statut** : 💡 idée · 📋 spécifié · 🚧 en cours · ✅ esquisse faite · 🔬 validé maquette · 🏭 PCB
**Complexité** : ⭐ très simple · ⭐⭐ core DIY · ⭐⭐⭐ puce dédiée · ⭐⭐⭐⭐ discret haut de gamme · ⭐⭐⭐⭐⭐ avancé/hybride

---

## A. Cœur — la matrice de routage CV (KiCad)

Tuiles I²C empilables. C'est le produit principal du dépôt.

| Module | Puce pivot | Rôle | Outil | Cplx | Statut |
|---|---|---|---|---|---|
| Tuile **Source** | MCP4728 (4× DAC 12 bit) | génère/buffer les CV, niveau par source | KiCad | ⭐⭐ | 📋 |
| Tuile **Crosspoint** | ADG2188 (8×8 I²C) | routage tenu on/off | KiCad | ⭐⭐⭐ | 📋 |
| Tuile **Destination** | TL074 sommateur + AD5254 digipot | mixe + master par bus + sortie | KiCad | ⭐⭐⭐ | 📋 |
| **Backplane / bus I²C** | TCA9548A (mux 8 sous-bus) | empilement, découverte au boot | KiCad | ⭐⭐ | 📋 |
| Maquette **tuile 8×8 bout-en-bout** | — | 1 source + 1 crosspoint + 1 dest | KiCad | ⭐⭐⭐ | 📋 |

## B. Infrastructure — alimentation, référence, frontières (Fritzing/KiCad)

| Module | Rôle | Outil | Cplx | Statut |
|---|---|---|---|---|
| **virtualGround** | masse virtuelle Vcc/2 (audio bipolaire sur alim simple) | Fritzing | ⭐ | ✅ |
| **Alimentation** ±12 / +5 / 3,3 V | rails + star ground (externe Eurorack ou embarquée) | KiCad | ⭐⭐ | 💡 |
| **Référence de tension** | réf. précise pour DAC / scaling | Fritzing | ⭐ | 💡 |
| **Entrée CV (frontière)** | scaling ±10 V → ±5 V + protection | Fritzing | ⭐⭐ | 💡 |
| **Sortie CV (frontière)** | scaling ±5 V → ±10 V + buffer | Fritzing | ⭐⭐ | 💡 |
| **ADC tap → NiDMI** | bus sommé resamplé → MIDI/OSC | KiCad | ⭐⭐ | 💡 |

## C. Sources de signal discrètes

> **Noise** et **S&H/Random** sont des _familles_ (cf. section D). Short-lists ordonnées
> par tier. Certaines variantes numériques recoupent le pont NiDMI (tuile Source) → à
> arbitrer hardware vs firmware.

### C.1 Noise — générateurs de bruit (famille)

| # | Variante | Tier | Principe / caractère | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **whiteNoiseGenerator** | 1 | avalanche B-E 2N3904 + 2 étages TL082 → blanc | Fritzing | ⭐ | ✅ |
| 2 | **noisePink** | 2 | blanc + filtre −3 dB/oct → rose | Fritzing | ⭐⭐ | 💡 |
| 3 | **noiseMultiColor** | 2 | blanc/rose/bleu par filtrage (type Buchla) | Fritzing | ⭐⭐ | 💡 |
| 4 | **noiseDigitalLFSR** | 3 | registre à décalage / µC → pseudo-aléatoire reproductible | KiCad/firmware | ⭐⭐⭐ | 💡 |

*(variante alternative T1 : bruit à diode Zener en avalanche au lieu du transistor)*

### C.2 Sample & Hold / Random — sources aléatoires (famille)

| # | Variante | Tier | Principe / caractère | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **sampleHold** | 1 | switch FET + capa + buffer, échantillonne au trig | Fritzing | ⭐ | 💡 |
| 2 | **randomCV** | 2 | bruit → S&H (+ slew pour random lissé) | Fritzing | ⭐⭐ | 💡 |
| 3 | **turingMachine** | 3 | registre à décalage bouclé → séquence aléatoire verrouillable | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **sourceOfUncertainty** | 5 | multi-section type Buchla 266 : random lissé/quantifié/stocké + distributions | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

## D. Voix, modulation & traitement (KiCad — backlog ambitieux)

> **VCO, VCF, EG, VCA et LFO sont des _familles_** : plusieurs variantes par topologie /
> caractère sonore, chacune dans son propre dossier `docs/modules/<variante>/`. Chaque
> short-list est ordonnée du plus simple au plus complexe (un tier de complexité par cran).

### D.1 VCO — oscillateurs (famille)

Short-list **validée**, ordonnée du plus simple au plus complet — parcours pédagogique
(un tier de complexité par cran, du proto Fritzing au discret KiCad puis à l'hybride
numérique piloté par NiDMI).

| # | Variante | Tier | Principe / caractère | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **vcoNE555** | 1 | timer NE555, CV sur pin 5 — carré/PWM, proto rapide | Fritzing | ⭐ | 💡 |
| 2 | **vcoOtaTriangle** | 2 | core triangle relaxation (LM13700/CA3080) — 1er vrai core analo | Fritzing | ⭐⭐ | 💡 |
| 3 | **vcoCEM3340** | 3 | core triangle intégré (CEM3340/AS3340/V3340) + tempco + sync | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **vcoMoogExpo** | 4 | saw core discret + **convertisseur expo** (paire appariée + tempco) + **waveshaper RSF** (tri→sinus) | KiCad | ⭐⭐⭐⭐ | 💡 |
| 5 | **vcoWavetable** | 5 | DDS µC + DAC, piloté NiDMI (pont numérique I²C) | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

> **Sous-blocs réutilisables du `vcoMoogExpo`** (candidats à isoler en modules/feuilles) :
> - **Convertisseur exponentiel** : paire appariée + compensation thermique (tempco) → V/oct.
> - **Waveshaper RSF** : mise en forme triangle → sinus (dérivé des conceptions RSF/Kobol).
> Ces deux blocs sont communs à plusieurs VCO → à factoriser.

### D.2 VCF — filtres (famille)

Short-list **validée**, ordonnée du plus simple au plus complexe (même logique que les
VCO : un tier de complexité par cran, du multimode accessible au ladder discret puis à
l'exotique vactrol).

| # | Variante | Tier | Principe / caractère | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **vcfSallenKeyMS20** | 2 | Sallen-Key VC, LP+HP 12 dB — caractère Korg MS-20 | Fritzing | ⭐⭐ | 💡 |
| 2 | **vcfStateVariable** | 3 | 2× OTA, LP/BP/HP/Notch simultanés — couteau suisse DIY | KiCad | ⭐⭐⭐ | 💡 |
| 3 | **vcfCEM3320** | 3 | puce OTA 4 pôles, pente réglable — Prophet/OB-Xa | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **vcfMoogLadder** | 4 | échelle à transistors 24 dB, auto-oscille — son Minimoog | KiCad | ⭐⭐⭐⭐ | 💡 |
| 5 | **vcfDiodeLadder** | 4 | échelle à diodes 24 dB — acid Roland TB-303 | KiCad | ⭐⭐⭐⭐ | 💡 |
| 6 | **vcfBuchlaLPG** | 5 | vactrol (LED+LDR), filtre+VCA couplés — West Coast | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

### D.3 EG — générateurs d'enveloppe (famille)

Short-list **validée**, ordonnée du plus simple au plus complexe (même logique que VCO/VCF :
un tier par cran, culminant sur le function generator façon Serge/Maths).

| # | Variante | Tier | Principe / caractère | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **egAR** | 1 | attack-release transistor + RC — proto, gate-suiveur | Fritzing | ⭐ | 💡 |
| 2 | **egADSR555** | 2 | ADSR discret (555/transistors + comparateurs) — DIY classique | Fritzing | ⭐⭐ | 💡 |
| 3 | **egCEM3310** | 3 | puce ADSR VC complète (CEM3310/AS3310) — Prophet/OB-Xa | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **egVCADSR** | 4 | ADSR entièrement commandé en tension (CV sur A/D/S/R) | KiCad | ⭐⭐⭐⭐ | 💡 |
| 5 | **egFunctionGen** | 5 | slope generator cyclable type Serge DUSG / Maths — multi-fonction | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

### D.4 VCA — amplis commandés en tension (famille)

Short-list **validée**, ordonnée du plus simple au plus complexe.

| # | Variante | Tier | Principe / caractère | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **vcaOTA** | 1 | OTA simple (LM13700) en VCA — son vintage, bruité | Fritzing | ⭐ | 💡 |
| 2 | **vcaVintageLM13700** | 2 | OTA + diodes de linéarisation + buffer, lin/exp — VCA DIY de référence | Fritzing | ⭐⭐ | 💡 |
| 3 | **vcaSSI2164** | 3 | quad VCA sur puce, lin/exp sans externes — propre, faible bruit | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **vcaDiscreteOTA** | 4 | cœur OTA discret, lin/exp commutable — type Crux | KiCad | ⭐⭐⭐⭐ | 💡 |

### D.5 LFO — oscillateurs basse fréquence (famille)

Short-list **validée**, ordonnée du plus simple au plus complexe. Le **`lfoICL8038`** est
prioritaire (puces ICL8038 en stock à écouler).

| # | Variante | Tier | Principe / caractère | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **lfo555** | 1 | astable 555 (+ intégrateur TL082) — carré/triangle, proto | Fritzing | ⭐ | 💡 |
| 2 | **lfoOpAmpMulti** | 2 | intégrateur + comparateur (+ sine shaper) — tri/carré/sinus | Fritzing | ⭐⭐ | 💡 |
| 3 | **lfoICL8038** | 3 | function generator 1 puce : sinus/triangle/carré natifs + sweep CV — **stock dispo** | Fritzing | ⭐⭐⭐ | 💡 |
| 4 | **lfoVCLFO** | 3 | rate commandé en tension, OTA — sorties simultanées multi-ondes | KiCad | ⭐⭐⭐ | 💡 |
| 5 | **lfoDigital** | 5 | µC + DAC, sync MIDI clock, formes arbitraires — piloté NiDMI | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

> ⚠️ Le `lfoDigital` **recoupe la tuile Source** de la matrice (spec §4 : « LFO internes »
> générés par l'ESP via DAC) → peut être **firmware** plutôt qu'un module hardware séparé.

### D.6 Autres traitements

| Module | Rôle | Outil | Cplx | Statut |
|---|---|---|---|---|
| **Slew limiter / portamento** | lissage de CV | Fritzing | ⭐ | 💡 |
| **Quantizer** | CV → notes (numérique, I²C) | KiCad | ⭐⭐ | 💡 |
| **Comparateur / fenêtre** | CV → gate/trig | Fritzing | ⭐ | 💡 |

## E. Utilitaires de patch (Fritzing)

| Module | Rôle | Outil | Cplx | Statut |
|---|---|---|---|---|
| **Attenuverter** | atténue + inverse une CV | Fritzing | ⭐ | 💡 |
| **Mult / buffered mult** | duplique une source | Fritzing | ⭐ | 💡 |
| **Mixer** | sommateur manuel | Fritzing | ⭐ | 💡 |
| **Offset / precision adder** | ajoute une tension fixe | Fritzing | ⭐ | 💡 |
| **Diviseur d'horloge** | sous-multiples de clock | Fritzing | ⭐ | 💡 |
| **Sortie ligne / casque** | sortie audio finale | Fritzing | ⭐⭐ | 💡 |

## F. Interfaces de contrôle & visualisation

> Surfaces de contrôle (entrée humaine) + visualisation, branchées sur **NiDMI/ESP32 en
> I²C** (cohérent « tout I²C » : ADS1115 pour les analogiques, MCP23017 pour boutons/LEDs,
> SSD1306/TFT pour l'écran). Plusieurs recoupent l'**UI/firmware NiDMI** (patchbay web,
> mapping) → arbitrer hardware vs firmware au cas par cas.

| Sous-famille | Contenu | Glue I²C typique | État |
|---|---|---|---|
| **F.1 Contrôleurs continus** | potards, faders, encodeurs | ADS1115 / mux 4067 | 📋 spécifié |
| **F.2 Contrôleurs gestuels** | pads (FSR/piézo/capacitif), ribbon, joystick, touch | ADC + MPR121 | 📋 spécifié |
| **F.3 Grilles & séquenceurs** | step sequencer, matrice de boutons, grille de pads RGB | MCP23017 / IS31FL3731 | 📋 spécifié |
| **F.4 Visualisation** | LED/RGB, bargraph, OLED/TFT, VU-mètre / mini-scope | SSD1306 / SPI TFT | 📋 spécifié |
| **F.5 Patchbay** | UI de la matrice de routage (recoupe la patchbay web, spec §5) | — (logiciel) | 📋 spécifié |

### F.1 Contrôleurs continus — potards (famille)

Short-list **validée**, ordonnée par méthode d'acquisition (c'est elle qui porte la
complexité). Le **Tier 3 (ADS1115/I²C)** est le sweet spot architectural recommandé.

| # | Variante | Tier | Acquisition / capacité | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **potBankDirect** | 1 | broches ADC ESP32 — ~6 potards, bruité | Fritzing | ⭐ | 💡 |
| 2 | **potBankMux** | 2 | mux CD4067 (16:1) → 1 ADC, 4 broches sélection | Fritzing | ⭐⭐ | 💡 |
| 3 | **potBankI2C** | 3 | ADS1115 (4 ch 16 bit) sur I²C, empilable — **base reco** | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **potBankMuxI2C** | 4 | CD4067 × ADS1115 → 64+ potards sur I²C | KiCad | ⭐⭐⭐⭐ | 💡 |
| 5 | **faderBankMotorized** | 5 | faders motorisés (double piste log+lin, touch-sense, H-bridge) → **rappel/motion** | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

> **Variantes transverses** (par fiche, orthogonales au tier) : taper lin (B) vs log (A),
> détente centrale (bipolaire), switch push, slide vs rotatif.
> **Alternative au T5** : encodeurs sans fin + anneau LED → rappel sans moteur (cf. encodeurs).

#### Encodeurs

Short-list **validée**, ordonnée par méthode de lecture. Le **Tier 2 (MCP23017)** est le
workhorse recommandé (pendant du `potBankI2C`).

| # | Variante | Tier | Lecture / capacité | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|
| 1 | **encoderBankGPIO** | 1 | quadrature directe sur GPIO ESP32 (décodage IRQ) — peu | Fritzing | ⭐ | 💡 |
| 2 | **encoderBankMCP23017** | 2 | expandeur MCP23017 I²C, IRQ — 8/puce, **jusqu'à 64** | KiCad | ⭐⭐ | 💡 |
| 3 | **encoderBankI2CRing** | 3 | encodeur I²C dédié + **anneau LED RGB** (DuPPa/Seesaw) → rappel d'état visuel | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **encoderAbsoluteAS5600** | 5 | magnétique **absolu 12 bit** (AS5600, I²C) — position vraie, sans homing | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

> **Variantes transverses** : mécanique à détente vs smooth, avec/sans push, optique, vélocité.
> Le **Tier 3 (anneau LED)** est l'alternative bon marché aux faders motorisés (`faderBankMotorized`).

### F.2 Contrôleurs gestuels — pads (famille)

Short-list **validée**, ordonnée par technologie de détection / expressivité
(on/off → vélocité → pression/aftertouch → position → multi-dimensionnel).

| # | Variante | Tier | Détection / expressivité | Glue | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|---|
| 1 | **padCapacitive** | 1 | touch on/off, 12 électrodes, sans bruit mécanique | MPR121 I²C | Fritzing | ⭐ | 💡 |
| 2 | **padPiezoVelocity** | 2 | vélocité de frappe (piézo), pas d'aftertouch — bon marché | ADC + clamp | Fritzing | ⭐⭐ | 💡 |
| 3 | **padFSRPressure** | 3 | vélocité + pression/aftertouch (FSR/Velostat), matrice muxée | ADS1115 / 4067 | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **ribbonSoftPot** | 4 | position + pression (SoftPot + FSR) → pitch + pressure + gate | ADC | KiCad | ⭐⭐⭐⭐ | 💡 |
| 5 | **padMPEgrid** | 5 | multi-touch position + pression par pad (type Trill/MPE) | capacitif + firmware | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

> **Velostat maison** remplace le FSR commercial (~8 $) à coût quasi nul.
> **Joystick** (2 axes) = transverse → 2 potards, relève de **F.1**.

### F.3 Grilles & séquenceurs (famille)

Short-list **validée**, ordonnée du séquenceur analogique pur au séquenceur firmware
(fond la grille d'interface — boutons + LEDs — et la logique de séquence).

| # | Variante | Tier | Principe | Glue | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|---|
| 1 | **seqAnalog4017** | 1 | séquenceur CD4017/4022 8 pas, 1 potard/pas → CV + gate, 1 LED/pas — zéro µC | logique CMOS | Fritzing | ⭐ | 💡 |
| 2 | **buttonMatrixLED** | 2 | matrice de boutons scannée (diodes) + LEDs simples | MCP23017 / ESP32 | Fritzing | ⭐⭐ | 💡 |
| 3 | **gridRGB** | 3 | grille boutons + LEDs RGB charlieplex (IS31FL3731 I²C, 16×9) — type Launchpad | IS31FL3731 I²C | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **padGridRGB** | 4 | grille de pads vélocité + RGB/pad (WS2812) — intègre F.2 | firmware | KiCad | ⭐⭐⭐⭐ | 💡 |
| 5 | **seqPerformer** | 5 | séquenceur firmware : CV/pas, ratchet, probabilité, rappel de patch NiDMI | firmware | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

> **seqPerformer** recoupe le firmware NiDMI et la patchbay (**F.5**) → arbitrer hardware vs logiciel.

### F.4 Visualisation (famille)

Short-list **validée**, ordonnée par quantité d'info affichée (indicateur → bargraph →
afficheur texte/graphique → scope → multifonction couleur).

| # | Variante | Tier | Affichage | Glue | Outil | Cplx | Statut |
|---|---|---|---|---|---|---|---|
| 1 | **ledIndicators** | 1 | LEDs gate/trig/clock, bicolore pour signe CV bipolaire | GPIO / MCP23017 | Fritzing | ⭐ | 💡 |
| 2 | **ledBargraph** | 2 | VU/niveau LM3914 (lin) / LM3915 (log dB), dot/bar — zéro µC | LM3914/15 | Fritzing | ⭐⭐ | 💡 |
| 3 | **oledSSD1306** | 3 | OLED 128×64 I²C, valeurs/menus/graphes — workhorse panneau | SSD1306 I²C | KiCad | ⭐⭐⭐ | 💡 |
| 4 | **miniScopeTFT** | 4 | mini-oscilloscope : ESP32 ADC + TFT ST7789 → forme d'onde/CV | TFT SPI + firmware | KiCad | ⭐⭐⭐⭐ | 💡 |
| 5 | **multiDisplayTFT** | 5 | TFT couleur multifonction (scope/tuner/spectral/clock) type Mordax DATA | TFT + firmware | KiCad | ⭐⭐⭐⭐⭐ | 💡 |

> **miniScopeTFT / multiDisplayTFT** recoupent le firmware NiDMI (ADC tap → affichage).

### F.5 Patchbay — contrôle du patch (modalités complémentaires)

⚠️ **Pas des variantes exclusives** : ces modalités agissent toutes sur le **même état de
patch** (persisté NVS). Ordonnées par effort hardware. Cf. spec §5 (UI patchbay) et §8.1
(« qui pilote le patch »).

| # | Variante | Tier | Modalité | Recoupe | Outil | Statut |
|---|---|---|---|---|---|---|
| 1 | **patchbayWeb** | 1 | UI web grille sources×destinations, clic = croisement, presets NVS — **prioritaire (spec §5)** | front NiDMI | firmware/web | 💡 |
| 2 | **patchbayNetwork** | 2 | rappel/pilotage MIDI-OSC (CC/PC → croisement ou snapshot) | spec §8.1 | firmware | 💡 |
| 3 | **patchbayEncoderOLED** | 2 | navigation encodeur + OLED : source→dest, toggle, niveaux | F.1 + F.4 | KiCad | 💡 |
| 4 | **patchbayButtonGrid** | 3 | grille physique de boutons illuminés (1/croisement, LED = fermé) — EMS-like électronique | F.3 `gridRGB` | KiCad | 💡 |

> Le **patch « vivant »** piloté par `MappingEngine` (scripts/signal) = **firmware NiDMI**,
> hors familles hardware. Les 4 modalités peuvent coexister sur le même patch NVS.

---

## Ordre suggéré

1. **Infrastructure d'abord** : Alimentation + virtualGround + Référence → tout le reste en dépend.
2. **Frontières CV** (entrée/sortie scaling) : indispensables dès qu'on branche du modulaire externe.
3. **Maquette tuile 8×8** : valider la matrice bout-en-bout (cf. spec §7).
4. **Modules discrets** au fil des besoins de patch (bruit ✅, S&H, attenuverter…).
5. **Voix** (VCO/VCF/VCA) en dernier — gros morceaux KiCad.
