# PlanÉlec

Outil de planification électrique résidentielle — fichier HTML unique, 100% local, sans dépendance externe.
sauvegarde du fichier en local, possibilité de le charger a nouveau.
disponible ici : https://vassily-fr.github.io/plan-elec/plan-electrique.html


---

## Table des matières

- [Concept](#concept)
- [Utilisation](#utilisation)
- [Architecture technique](#architecture-technique)
- [Fonctionnalités détaillées](#fonctionnalités-détaillées)
- [Normes appliquées (NF C 15-100)](#normes-appliquées-nf-c-15-100)
- [État de l'application (State)](#état-de-lapplication-state)
- [Roadmap / Améliorations prévues](#roadmap--améliorations-prévues)
- [Bugs connus](#bugs-connus)

---

## Concept

Logiciel HTML/JS monofichier permettant de :

1. Charger une image de plan d'architecte (JPG, PNG, etc.)
2. Poser des éléments électriques par glisser-déposer
3. Tracer les chemins de gaines sur le plan
4. Définir une échelle réelle pour calculer les longueurs en mètres
5. Gérer plusieurs étages indépendants
6. Obtenir un bilan d'achat (quantités) et un bilan électrique (circuits, disjoncteurs)
7. Assigner chaque élément à un circuit manuellement ou via une répartition automatique NF C 15-100
8. Sauvegarder/charger le projet en JSON local

**Stack :** HTML5 + CSS3 + Canvas 2D + JS vanilla. Aucune lib externe (polices Google Fonts en CDN uniquement).

---

## Utilisation

### Lancement

Ouvrir `plan-electrique.html` directement dans un navigateur moderne (Chrome, Firefox, Edge). Aucun serveur requis.

### Workflow type

1. **Charger le plan** — bouton "Charger plan" dans la barre gauche ou glisser-déposer une image sur le canevas
2. **Définir l'échelle** — outil "Échelle" → tracer une ligne sur le plan → entrer la longueur réelle en mètres
3. **Placer les éléments** — glisser-déposer depuis la barre gauche vers le plan
4. **Tracer les gaines** — outil "Ligne droite" (deux clics) ou "Gaine libre" (polylignes, double-clic pour terminer)
5. **Gérer les circuits** — onglet "Circuits" → bouton "Répartition automatique" puis affiner manuellement
6. **Consulter le bilan** — onglet "Bilan" pour les quantités et longueurs de câbles
7. **Ajouter des étages** — bouton "＋" dans le header, recommencer depuis l'étape 1
8. **Sauvegarder** — bouton "💾 Sauvegarder" → fichier `plan-electrique.json`

### Raccourcis clavier

| Touche | Action |
|--------|--------|
| `Échap` | Annuler l'outil en cours (gaine, ligne, échelle) |
| `Entrée` | Terminer la gaine libre en cours |
| `Suppr` / `Retour arrière` | Supprimer l'élément sélectionné |
| `Dbl-clic` (canevas) | Terminer la gaine libre en cours |

### Clic droit sur un élément

- Pivoter de 45°
- Dupliquer
- Supprimer

---

## Architecture technique

### Structure du fichier

```
plan-electrique.html
├── <style>        CSS inline — variables CSS + layout flex + composants
├── <body>         HTML structure
│   ├── #hdr       Header : titre, onglets étages, boutons save/load
│   ├── #tb        Toolbar gauche : éléments + outils
│   ├── #ca        Zone canevas (canvas 2D)
│   └── #rp        Panel droit : Bilan / Circuits / Réglages
└── <script>       JS vanilla inline
    ├── T{}        Définitions des types d'éléments
    ├── S{}        État global de l'application
    ├── Niveaux    addLvl / switchLvl / delCurLvl / rLvlTabs
    ├── Canvas     setupCV / render / drawEl / elAt / placeEl / delEl
    ├── Outils     setTool + handlers mouse (select/line/wire/scale)
    ├── Panels     renderBP (bilan) / renderCP (circuits) / setupSettings
    ├── Circuits   autoAssign (NF C 15-100) + gestion manuelle
    └── Save/Load  JSON export/import
```

### Types d'éléments (`T{}`)

```js
{
  prise16:      { label, color:'#f0c040', cat:'prise',   amp:16, sec:'2.5mm²' },
  prise20:      { label, color:'#ff9a3c', cat:'prise20', amp:20, sec:'2.5mm²' },
  inter_simple: { label, color:'#5b8cff', cat:'inter',   amp:10, sec:'1.5mm²' },
  inter_double: { label, color:'#5b8cff', cat:'inter',   amp:10, sec:'1.5mm²' },
  luminaire:    { label, color:'#4ecdc4', cat:'lumiere', amp:10, sec:'1.5mm²' },
  tableau:      { label, color:'#ff6b6b', cat:'tableau', amp:0,  sec:'-'       },
}
```

### État global (`S{}`)

```js
S = {
  lvls: [           // tableau des étages
    {
      id,           // float unique (Date.now() + random)
      name,         // string
      img,          // base64 dataURL ou null
      els: [        // éléments posés
        { id, type, x, y, rot }
      ],
      wires: [      // gaines tracées
        [ {x,y}, {x,y}, ... ]  // tableau de points
      ]
    }
  ],
  curLvl,           // index de l'étage actif
  scale,            // pixels par mètre (null si non défini)
  tool,             // 'select' | 'line' | 'wire' | 'scale'
  drag,             // référence à l'élément en cours de déplacement
  dragOff,          // {x,y} offset souris/élément
  wirePs,           // [{x,y}...] points de la gaine en cours
  linePt,           // {x,y} premier point de la ligne droite en cours
  scaleS,           // {x1,y1,x2?,y2?,done?} segment d'échelle en cours
  selEl,            // élément sélectionné (référence)
  ctxEl,            // élément du menu contextuel
  mx, my,           // position souris courante
  circs: [          // circuits définis
    { id, name, disj, sec, type }
  ],
  assgn: {},        // { elId: circId } assignation éléments → circuits
  cfg: {
    margin,         // % de marge ajoutée aux longueurs (défaut 10)
    maxP16,         // prises 16A max par circuit (défaut 8, max NFC: 8)
    maxLum,         // lumineux max par circuit (défaut 8, max NFC: 8)
  }
}
```

### Format de sauvegarde JSON

```json
{
  "v": 2,
  "scale": 42.5,
  "cfg": { "margin": 10, "maxP16": 8, "maxLum": 8 },
  "circs": [
    { "id": 1234567, "name": "Prises 16A-1", "type": "prise16", "disj": 16, "sec": "2.5mm²" }
  ],
  "assgn": {
    "1234568": "1234567"
  },
  "lvls": [
    {
      "id": 1234569,
      "name": "RDC",
      "img": "data:image/png;base64,...",
      "els": [
        { "id": 1234570, "type": "prise16", "x": 320, "y": 214, "rot": 0 }
      ],
      "wires": [
        [ {"x": 100, "y": 200}, {"x": 300, "y": 200} ]
      ]
    }
  ]
}
```

> ⚠️ Les images sont stockées en base64 dans le JSON — les fichiers peuvent être lourds (~3–5× la taille de l'image originale).

---

## Fonctionnalités détaillées

### Modes de tracé de gaines

**Ligne droite** (`tool: 'line'`) :
- Clic 1 → point de départ
- Clic 2 → point d'arrivée → segment enregistré immédiatement
- Idéal pour les tracés rectilignes le long des murs

**Gaine libre** (`tool: 'wire'`) :
- Chaque clic ajoute un point à la polyligne en cours
- Double-clic ou `Entrée` → termine et enregistre
- Clic droit ou `Échap` → termine si ≥ 2 points, annule sinon
- Idéal pour suivre des murs non rectilignes

Les deux modes alimentent le même tableau `wires` et contribuent au même calcul de longueur.

### Mise à l'échelle

- L'outil "Échelle" demande de tracer une ligne sur le plan
- Après confirmation de la longueur réelle, `S.scale` est calculé : `scale = longueurPixels / longueurMètres`
- Toutes les longueurs de gaines sont ensuite converties : `mètres = pixels / S.scale`
- Un point coloré est affiché sur chaque élément indiquant son circuit assigné

### Calcul des longueurs de câbles

La longueur totale tracée (`Σ segments`) est multipliée par `(1 + margin/100)`.

La répartition par section est une **estimation proportionnelle** basée sur les éléments présents :
- 30% de la longueur → fils 1.5mm² (si des lumineux ou interrupteurs sont présents)
- 60% → fils 2.5mm² (si des prises 16A ou 20A sont présentes)

> Cette estimation est délibérément simplifiée. Une amélioration future (voir Roadmap) permettrait de calculer précisément par circuit.

### Répartition automatique des circuits

La fonction `autoAssign()` applique NF C 15-100 :

```
Prises 16A  → groupes de maxP16 (≤8) → 1 circuit = 1 disjoncteur 16A 2.5mm²
Prises 20A  → 1 par circuit           → 1 circuit = 1 disjoncteur 20A 2.5mm²
Luminaires  → groupes de maxLum (≤8)  → 1 circuit = 1 disjoncteur 10A 1.5mm²
Interrupteurs → associés aux circuits éclairage correspondants (répartition linéaire)
```

Après l'auto-assign, l'utilisateur peut :
- Renommer n'importe quel circuit (champ texte inline)
- Supprimer un circuit (les éléments deviennent "non assignés")
- Ajouter des circuits manuellement
- Réassigner individuellement chaque élément via sa liste déroulante

### Identification visuelle des circuits

Chaque circuit reçoit une couleur de la palette `CC[]` :
```js
const CC = ['#5b8cff','#5ddb8e','#ff9a3c','#e879f9','#f0c040','#4ecdc4','#ff85a1','#a3e635'];
```
Un petit disque coloré s'affiche en haut à droite de chaque élément assigné sur le canevas.

---

## Normes appliquées (NF C 15-100)

| Type | Section | Disjoncteur | Max par circuit |
|------|---------|-------------|-----------------|
| Prise 16A standard | 2.5mm² | 16A | 8 |
| Prise 20A spécialisée (cuisine, four, lave-linge) | 2.5mm² | 20A | 1 |
| Éclairage / interrupteurs | 1.5mm² | 10A | 8 points lumineux |
| Arrivée principale | 10mm² | selon puissance | — |

Le tableau électrique doit prévoir : `N circuits individuels + 1 disjoncteur différentiel 30mA`.

---

## Roadmap / Améliorations prévues

### Priorité haute

- [ ] **Calcul précis des fils par circuit** — au lieu d'une estimation proportionnelle, calculer la longueur des gaines associées à chaque circuit (nécessite de lier les gaines aux circuits)
- [ ] **Association gaine ↔ circuit** — permettre de colorier les gaines selon le circuit auquel elles appartiennent
- [ ] **Export PDF** — générer un récapitulatif imprimable (bilan + plan annoté)
- [ ] **Annuler/Rétablir** (Ctrl+Z / Ctrl+Y) — historique des actions

### Priorité moyenne

- [ ] **Zoom + pan du canevas** — actuellement l'image est étirée à la taille du canevas, un zoom serait plus précis
- [ ] **Snap à la grille** — pour aligner les éléments et faciliter la mise à l'échelle
- [ ] **Étiquettes personnalisées** sur les éléments (ex : "Prise cuisine droite")
- [ ] **Prise 32A (cuisson)** — supprimée en v2 par choix utilisateur, à réintégrer en option
- [ ] **Spots encastrés** — idem
- [ ] **Export image** du plan annoté (canvas.toBlob)
- [ ] **Copier-coller** d'éléments (Ctrl+C / Ctrl+V)

### Priorité basse

- [ ] **Mode sombre/clair** toggle
- [ ] **Bibliothèque de plans** — gérer plusieurs projets/maisons dans le même fichier
- [ ] **Vue synthèse multi-étages** — voir tous les étages côte à côte
- [ ] **Calcul de la puissance souscrite** recommandée (basé sur les circuits)
- [ ] **Mode tablette/touch** — support des événements touch pour usage sur tablette

---

## Bugs connus

- **Images lourdes** : les grandes images (>5Mo) peuvent ralentir le rendu car recalculées à chaque frame. Mitigation : `bgC{}` cache les objets `Image`, mais le `drawImage` reste appelé à chaque `render()`.
- **IDs flottants** : les IDs utilisent `Date.now() + Math.random()` — peu de risque de collision mais pas idéal. À remplacer par un UUID ou un compteur auto-incrémenté.
- **Assignation perdue au rechargement JSON** : si les IDs d'éléments ne correspondent plus (modification manuelle du JSON), les assignations `assgn{}` pointent vers des éléments inexistants sans erreur visible.
- **Gaines non reliées aux circuits** : les gaines sont indépendantes des circuits, le calcul de longueur par section est donc une approximation.

---

## Notes pour la reprise avec Claude Code

- Tout le code est dans un seul fichier `plan-electrique.html` — CSS, HTML et JS inline
- L'état est dans l'objet global `S` — pas de framework, pas de réactivité automatique, les panels sont re-rendus manuellement via `upPanels()` / `render()`
- Le canevas est redessiné entièrement à chaque `render()` (pas de layers)
- Pour refactoriser : commencer par extraire JS dans `app.js` et CSS dans `style.css`, puis éventuellement migrer vers un bundler (Vite) si le projet grossit
- Les fonctions clés à connaître : `render()`, `upPanels()`, `autoAssign()`, `renderCP()`, `renderBP()`
