# PlanÉlec

Outil de planification électrique résidentielle : un seul fichier HTML, 100 % local, sans dépendance externe.
Le projet se sauvegarde automatiquement (navigateur, et fichier JSON local s'il y en a un ouvert) et peut être rechargé.
Disponible ici : https://vassily-fr.github.io/plan-elec/plan-electrique.html

---

## Table des matières

- [Concept](#concept)
- [Utilisation](#utilisation)
- [Liaisons de commande](#liaisons-de-commande)
- [Mode impression](#mode-impression)
- [Check-list chantier](#check-list-chantier)
- [Architecture technique](#architecture-technique)
- [Fonctionnalités détaillées](#fonctionnalités-détaillées)
- [Normes appliquées (NF C 15-100)](#normes-appliquées-nf-c-15-100)
- [Roadmap / Améliorations prévues](#roadmap--améliorations-prévues)
- [Bugs connus](#bugs-connus)
- [Notes pour la reprise avec Claude Code](#notes-pour-la-reprise-avec-claude-code)

---

## Concept

Un logiciel HTML/JS en un seul fichier qui permet de :

1. Charger l'image d'un plan d'architecte (JPG, PNG, SVG…).
2. Poser des éléments électriques par glisser-déposer : prises, interrupteurs, éclairage, volets, réseau, tableau.
3. Relier les commandes à ce qu'elles commandent (interrupteur → lumière, inter. volet → volet, va-et-vient).
4. Tracer les gaines, chacune rattachée à un circuit.
5. Définir une échelle réelle pour calculer les longueurs en mètres.
6. Gérer plusieurs étages indépendants.
7. Obtenir un bilan d'achat (quantités, mécaniques, câbles) et un bilan électrique (circuits, disjoncteurs).
8. Répartir les éléments en circuits, à la main ou automatiquement selon la NF C 15-100.
9. Imprimer un plan d'électricien (symboles normalisés, aimant d'alignement, légende, cartouche) et une check-list de chantier.
10. Sauvegarder et recharger le projet en JSON local.

**Stack :** HTML5, CSS3, Canvas 2D et JS vanilla. Aucune bibliothèque externe : seules les polices Google Fonts viennent d'un CDN.

---

## Utilisation

### Lancement

Ouvrir `plan-electrique.html` dans un navigateur moderne (Chrome, Edge, Firefox, Safari). Aucun serveur n'est requis.

Un projet d'exemple est fourni : `test-3-circuits.json` (3 circuits, interrupteurs reliés, volets).

### Workflow type

1. **Charger le plan** : bouton « Charger plan » ou glisser-déposer une image sur le canevas.
2. **Définir l'échelle** : outil « Échelle », tracer une ligne sur le plan, puis entrer sa longueur réelle en mètres.
3. **Placer les éléments** : glisser-déposer depuis la barre de gauche.
4. **Relier les commandes** : clic droit sur un interrupteur, puis « 🔗 Relier », puis clic sur la lumière (voir [Liaisons](#liaisons-de-commande)).
5. **Gérer les circuits** : onglet « Circuits », bouton « ⚡ Répartition automatique », puis affiner à la main.
6. **Tracer les gaines** : choisir le circuit actif, puis utiliser « Ligne droite » (deux clics) ou « Gaine libre » (polyligne).
7. **Renseigner les hauteurs** : clic droit, puis « 📏 Hauteur » (valeurs par défaut fournies).
8. **Consulter le bilan** : onglet « Bilan » pour les quantités, les mécaniques et les longueurs de câbles.
9. **Ajouter des étages** : bouton « ＋ » dans l'en-tête.
10. **Imprimer** : bouton « 🖨 Impression » (plans et/ou check-list, un ou tous les étages).
11. **Sauvegarder** : automatique (voir ci-dessous) ; « 💾 Sauvegarder » ou Ctrl/⌘+S pour écrire le fichier JSON.

### Sauvegarde automatique

- Chaque modification est enregistrée environ 1 s plus tard dans le navigateur (IndexedDB), et le dernier projet est rechargé à la prochaine ouverture de la page.
- Si le projet vient d'un fichier JSON (ouvert ou sauvegardé via 💾), ce fichier est aussi réécrit automatiquement, dès que le navigateur a donné l'accord d'écriture. Après un rechargement de la page, Chrome redemande cet accord : un clic sur 💾 suffit, l'écriture redevient ensuite automatique.
- L'indicateur à droite de l'en-tête montre l'état : « ● enregistrement… », « ✓ auto HH:MM » (navigateur), « ✓ fichier.json · HH:MM », ou « fichier à mettre à jour (Ctrl+S) ».
- La sauvegarde navigateur dépend du site (une copie par adresse) et ne remplace pas un fichier JSON pour archiver ou changer d'ordinateur.
- **📄 Nouveau** repart d'un projet vide.

### Renommer

- **Éléments** : double-clic sur l'élément, clic droit puis « ✏ Renommer », ou double-clic sur son nom dans la liste de l'onglet Circuits. Vide = nom par défaut. Le nom s'affiche sous le picto, dans les infobulles, la liste des circuits et la check-list.
- **Étages** : double-clic sur l'onglet de l'étage.
- **Circuits** : champ de nom de la carte du circuit (onglet Circuits).

### Navigation dans le plan

| Action | Geste |
|--------|-------|
| Zoomer | Pincer, Ctrl+molette, touches `+` / `−`, boutons − / + |
| Recadrer | Touche `0`, bouton ⤢ ou clic sur le pourcentage |
| Déplacer | Glisser dans le vide, Espace+glisser, ou molette |

Le zoom et le déplacement ne sont pas sauvegardés.

### Raccourcis clavier

| Touche | Action |
|--------|--------|
| `Ctrl/⌘+Z` | Annuler (historique des actions) |
| `Ctrl/⌘+S` | Sauvegarder |
| `Échap` | Annuler l'outil en cours (gaine, ligne, échelle, liaison) ; quitte le mode impression |
| `Entrée` / double-clic | Terminer la gaine libre en cours |
| `Suppr` / `Retour arrière` | Supprimer l'élément ou la gaine sélectionnés |
| `+` `−` `0` `Espace` | Zoom, dézoom, recadrer, déplacer |

### Clic droit sur un élément

- 🔄 Pivoter de 45°
- ✏ Renommer (aussi par double-clic sur l'élément)
- 📋 Dupliquer
- 🔗 Relier (interrupteurs et inter. volets)
- ✂ Délier : retire **toutes** les liaisons de l'élément en un clic
- 📏 Hauteur : hauteur de pose en cm ; laisser vide pour revenir à la valeur par défaut
- Choix du circuit
- Mécaniques (prise double : élec / RJ45 / HDMI…)
- 🗑 Supprimer

Un clic droit sur une gaine propose 🗑 Supprimer gaine.

---

## Liaisons de commande

- Une liaison relie une commande (interrupteur, double / va-et-vient, inter. volet) à ce qu'elle commande (luminaire, applique, volet). Elle est stockée sur l'élément : `el.links = [{ id, pts }]`, où `pts` sont des points de passage optionnels.
- **Sélection** : sélectionner un élément affiche et met en évidence **tout son réseau**, recherché en largeur par `linkGroup`. Par exemple, en va-et-vient, cliquer la lumière montre les deux interrupteurs.
- **Délier** : `unlinkAll(el)` supprime d'un coup toutes les liaisons de l'élément, dans les deux sens. Un bandeau confirme le nombre de liaisons retirées. Une liaison retirée par erreur se refait simplement.

---

## Mode impression

Le bouton **🖨 Impression** passe en aperçu papier, inspiré des plans d'architecte et d'électricien. Le mode édition reste inchangé.

### Plan

- Le plan d'architecte s'affiche en fond, en niveaux de gris et éclairci : réglage « Plan », de 0 à 100 %.
- Les symboles sont normalisés, en noir :
  - prise : demi-cercle avec trait de terre ;
  - interrupteur : cercle et bras ; va-et-vient : deux bras opposés ;
  - luminaire : cercle barré d'une croix ; applique : avec barre murale ;
  - tableau : rectangle demi-plein ;
  - VR, VB, RJ et inter. volet (flèches) : symboles dédiés.
- **Pictos** : taille réglable en mm papier (4 mm par défaut).
- **Aimant** : les éléments muraux proches sont regroupés, par union-find avec un seuil en mm papier réglable, puis alignés en rangée le long de leur axe principal. On obtient ainsi les « blocs » d'un vrai plan d'électricien.
- **Commandes** : arcs pointillés entre chaque interrupteur et ce qu'il commande.
- **Repères** : numérotation par type (P1, I3, L2, IV1, V1, RJ1…) dans l'ordre de lecture, avec la hauteur de pose.
- Options **Gaines** (tracés des gaines) et **Couleurs** (couleur par circuit) ; étiquettes de circuit C1, C2…
- Cadre, légende (symboles présents avec leur nombre, liste des circuits) et cartouche : projet, étage, date, échelle ≈ 1:N, barre d'échelle.
- Formats A4 / A3, paysage / portrait.

### Choix des pages

- **Contenu** : « Plans + check-list », « Plans seuls » ou « Check-list seule ».
- **Étages** : « Tous les étages » (ceux qui ont un plan ou des éléments) ou l'étage courant. Le sélecteur n'apparaît que s'il y a plusieurs étages.
- Les pages sortent étage par étage : le plan de l'étage, puis sa check-list.
- L'aperçu s'ouvre sur l'étage affiché ; ◀ ▶ parcourt toutes les pages (numéro et étage affichés).
- **🖨 Imprimer sort exactement les pages de l'aperçu.**

### Technique

- Le rendu ne dépend pas de la résolution : la page est définie en mm, et `k` vaut le nombre de px par mm.
  - `drawSheet(g, k, o, lvl)` dessine le plan d'un étage.
  - `drawChecklist(g, k, o, page, pages, lvl)` dessine une page de check-list d'un étage.
  - `printPages(o)` liste les pages à produire (`{l, kind, draw}`), selon `o.what` et `o.lvls`.
  - Avant l'impression, `loadPlanImg()` attend le chargement des plans des étages non encore affichés.
- À l'impression, chaque page est rendue hors écran à k = 8 (≈ 200 dpi), convertie en PNG et placée dans `#printOut`. Une règle `@page` est injectée avec le bon format et la bonne orientation, puis `window.print()` est appelé.
- Les options sont sauvegardées dans le projet (`cfg.print`) et fusionnées avec les valeurs par défaut `PDEF` par `pc()`.

---

## Check-list chantier

Elle est faite pour être imprimée et cochée au crayon sur le chantier, avec des gants et sans ordinateur. On voit d'un coup d'œil ce qui reste à faire.

- **Une check-list par étage** : l'en-tête affiche le nom de l'étage en cartouche noir, repris dans le pied de page. Les repères (P1, I2…) sont numérotés par étage, comme sur le plan de cet étage.
- **Une ligne par point** : prise, interrupteur, lumière, volet, prise réseau. Dans l'étage, les lignes sont groupées par circuit, puis viennent les points « Sans circuit ».
- **Colonnes** :
  - Repère (picto et numéro, identiques au plan) ;
  - Point (nom personnalisé en gras s'il existe, puis type et mécaniques) ;
  - Circuit ;
  - **H (cm)** : hauteur de pose, pour retrouver les boîtes à la scie cloche ;
  - 7 cases à cocher ;
  - Notes.
- **Étapes** :

  | Case | Signification |
  |------|---------------|
  | Boîte | Boîte d'encastrement posée |
  | Gaine | Gaine passée jusqu'au point |
  | Repérée | Gaine identifiée / étiquetée aux deux bouts |
  | Fils | Fils tirés |
  | Méca | Mécanisme posé |
  | Tableau | Raccordé au tableau |
  | Testé | Essai fonctionnel OK |

  Les volets n'ont ni case « Boîte » ni case « Méca » (cases grisées).
- Chaque en-tête de circuit porte une case « Disjoncteur posé et étiqueté ».
- La pagination évite qu'un en-tête de circuit reste seul en bas de page. En A4 portrait, on compte environ 29 lignes par page.

### Hauteurs par défaut (cm, NF C 15-100 / usages)

| Type | Hauteur |
|------|---------|
| Prises 16A, 20A, double, réseau | 30 |
| Interrupteurs, inter. volet | 110 |
| Applique | 180 |
| Luminaire plafond, tableau | — |

Une hauteur personnalisée est stockée dans `el.h`. Elle s'affiche aussi dans l'info-bulle de l'élément (« · h X cm »).

---

## Architecture technique

### Structure du fichier

```
plan-electrique.html
├── <style>        CSS inline : variables CSS, layout flex, composants, @media print
├── <body>
│   ├── #hdr       En-tête : titre, onglets d'étages, sauvegarder / charger, 🖨 Impression
│   ├── #tb        Barre gauche : éléments et outils
│   ├── #ca        Zone canevas (canvas 2D), zoom, barre d'impression #pbar
│   ├── #rp        Panneau droit : Bilan / Circuits / Réglages
│   ├── #ctx       Menu contextuel
│   └── #printOut  Pages PNG générées pour l'impression
└── <script>       JS vanilla inline
    ├── T{}        Types d'éléments
    ├── S{}        État global
    ├── Niveaux    addLvl / switchLvl / delCurLvl / rLvlTabs
    ├── Vue        imgXf / w2c / c2w / zoomAt / resetView
    ├── Canvas     render / drawEl / elAt / placeEl / delEl
    ├── Liaisons   startLink / endLink / linkedEls / linkGroup / unlinkAll
    ├── Outils     setTool + gestionnaires souris (select / line / wire / scale)
    ├── Panneaux   renderBP (bilan) / renderCP (circuits) / setupSettings
    ├── Circuits   autoAssign (NF C 15-100) + gestion manuelle
    ├── Historique pushHistory / undo
    ├── Impression setupPrint / renderPrint / drawSheet / drawChecklist / doPrint
    └── Save/Load  projectData / saveProject / loadData (File System Access si dispo)
```

### Types d'éléments (`T{}`)

| Clé | Libellé | Catégorie | Disj. | Section |
|-----|---------|-----------|-------|---------|
| `prise16` | Prise 16A | prise | 16A | 2.5mm² |
| `prise_double` | Prise double (mécas : élec, RJ45, HDMI…) | prise | 16A | 2.5mm² |
| `prise20` | Prise 20A | prise20 | 20A | 2.5mm² |
| `inter_simple` | Interrupteur | inter | 10A | 1.5mm² |
| `inter_double` | Double / va-et-vient | inter | 10A | 1.5mm² |
| `luminaire` | Luminaire | lumiere | 10A | 1.5mm² |
| `applique` | Applique murale | lumiere | 10A | 1.5mm² |
| `volet_roulant` | Volet roulant | volet | 10A | 1.5mm² |
| `volet_battant` | Volet battant | volet | 10A | 1.5mm² |
| `inter_volet` | Inter. volet | inter_volet | 10A | 1.5mm² |
| `prise_rj45` | Prise réseau | reseau | — | — |
| `tableau` | Tableau | tableau | — | — |

### État global (`S{}`)

```js
S = {
  lvls: [{ id, name, img, els: [...], wires: [...] }],
  curLvl,            // index de l'étage actif
  scale,             // pixels par mètre (null si non définie)
  tool,              // 'select' | 'line' | 'wire' | 'scale'
  selEl, ctxEl,      // élément sélectionné / du menu contextuel
  selWire,           // index de la gaine sélectionnée (-1 = aucune)
  linkFrom, linkPts, // liaison en cours de tracé
  focusCirc,         // circuit mis en avant
  activeCircId,      // circuit actif pour le tracé de gaines
  circs: [{ id, name, disj, sec, type }],
  assgn: {},         // { elId: circId }
  hiddenCircs: {},   // circuits masqués
  hiddenRouting: {}, // circuits dont seul le routage est masqué
  cfg: { margin, maxP16, maxLum, planAlpha, planInv, print: {...} },
  view: { z, x, y }, // zoom / déplacement (non sauvegardé)
  hist: [],          // historique d'annulation (instantanés)
  printMode, pg,     // mode impression et page affichée
  fileHandle,        // fichier ouvert (File System Access API)
}
```

### Format de sauvegarde JSON (v5)

```json
{
  "v": 5,
  "scale": 100,
  "cfg": {
    "margin": 10, "maxP16": 8, "maxLum": 8, "planAlpha": 80, "planInv": false,
    "print": { "page": "plan", "fmt": "A3", "ori": "L", "sym": 4, "mag": true, "magD": 8,
               "plan": 30, "lnk": true, "rep": true, "wir": false, "col": false }
  },
  "circs": [{ "id": 1, "name": "Éclairage-1", "type": "lumiere", "disj": 10, "sec": "1.5mm²" }],
  "assgn": { "5": 1 },
  "hiddenCircs": {}, "hiddenRouting": {},
  "lvls": [{
    "id": 24, "name": "RDC", "img": "data:image/png;base64,...",
    "els": [
      { "id": 5, "type": "luminaire", "x": 350, "y": 250, "rot": 0 },
      { "id": 9, "type": "inter_double", "x": 90, "y": 420, "rot": 0, "h": 110,
        "links": [{ "id": 5, "pts": [{ "x": 90, "y": 250 }] }] },
      { "id": 16, "type": "prise_double", "x": 400, "y": 435, "rot": 0, "meca1": "elec", "meca2": "rj45" }
    ],
    "wires": [{ "pts": [{ "x": 100, "y": 700 }, { "x": 70, "y": 150 }], "circId": 2 }]
  }]
}
```

Champs facultatifs d'un élément : `name` (nom personnalisé), `h` (hauteur en cm), `links`, `meca1` / `meca2`.
Les anciens formats (v2 à v4) restent chargeables.

> ⚠️ Les images sont stockées en base64 dans le JSON : le fichier pèse environ 1,3 à 5 fois l'image d'origine.

---

## Fonctionnalités détaillées

### Modes de tracé de gaines

**Ligne droite** (`tool: 'line'`) : le premier clic fixe le départ, le second l'arrivée.

**Gaine libre** (`tool: 'wire'`) : chaque clic ajoute un point. Double-clic ou `Entrée` termine la gaine ; `Échap` l'annule.

Chaque gaine est rattachée au circuit actif (`circId`) et prend sa couleur. On peut masquer les gaines circuit par circuit.

### Mise à l'échelle

On trace une ligne de longueur connue, puis `S.scale = longueurPixels / longueurMètres`. Toutes les longueurs sont ensuite converties en mètres, majorées de `cfg.margin` %.

### Bilan

- Quantités par type d'élément et nombre de mécaniques, prises doubles comprises.
- Longueur de gaine par circuit, donc par section de câble, y compris les volets.

### Répartition automatique des circuits

`autoAssign()` applique la NF C 15-100 :

```
Prises 16A     → groupes de maxP16 (≤ 8) → disjoncteur 16A, 2.5mm²
Prises 20A     → 1 par circuit            → disjoncteur 20A, 2.5mm²
Éclairage      → groupes de maxLum (≤ 8)  → disjoncteur 10A, 1.5mm²
Interrupteurs  → circuit de la lumière qu'ils commandent
Volets         → circuit volets 10A, 1.5mm²
```

On peut ensuite renommer, ajouter ou supprimer des circuits, réassigner un élément, et masquer ou mettre en avant un circuit. Sélectionner un élément sélectionne aussi son circuit dans la liste.

### Identification visuelle

Chaque circuit reçoit une couleur de la palette `CC[]`, affichée en pastille sur l'élément et sur ses gaines. Réglages du fond de plan : opacité (`planAlpha`) et inversion (`planInv`).

---

## Normes appliquées (NF C 15-100)

| Type | Section | Disjoncteur | Max par circuit |
|------|---------|-------------|-----------------|
| Prise 16A standard | 2.5mm² | 16A | 8 |
| Prise 20A spécialisée (four, lave-linge…) | 2.5mm² | 20A | 1 |
| Éclairage / interrupteurs | 1.5mm² | 10A | 8 points lumineux |
| Volets roulants | 1.5mm² | 10A | — |
| Arrivée principale | 10mm² | selon puissance | — |

Le tableau doit prévoir N circuits individuels, protégés par des interrupteurs différentiels 30 mA.

Hauteurs usuelles : prises à 30 cm (5 cm minimum), commandes entre 90 et 130 cm (110 cm par défaut).

---

## Roadmap / Améliorations prévues

### Fait

- [x] Annuler (Ctrl+Z) avec historique
- [x] Zoom et déplacement du canevas
- [x] Association gaine ↔ circuit, et longueurs par circuit
- [x] Liaisons interrupteur ↔ lumière / volet, va-et-vient
- [x] Volets roulants / battants, prise double, prise réseau, applique
- [x] Impression du plan d'électricien et check-list de chantier (PDF possible via la fenêtre d'impression)
- [x] Hauteur de pose par élément
- [x] Sauvegarde automatique (navigateur + fichier JSON ouvert)
- [x] Noms personnalisés des éléments (ex. « Prise cuisine droite »), repris à l'écran et sur la check-list
- [x] Impression au choix : plans, check-list ou les deux, un étage ou tous ; check-list par étage

### Priorité moyenne

- [ ] Rétablir (Ctrl+Y)
- [ ] Check-list cochable aussi dans l'appli (suivi de l'avancement sauvegardé)
- [ ] Bilan imprimable (liste d'achat)
- [ ] Prise 32A (cuisson), spots encastrés
- [ ] Copier-coller (Ctrl+C / Ctrl+V)

### Priorité basse

- [ ] Vue synthèse multi-étages
- [ ] Puissance souscrite recommandée
- [ ] Support tactile complet (tablette)

---

## Bugs connus

- **Images lourdes** : les grandes images (> 5 Mo) alourdissent le JSON et le rendu.
- **IDs flottants** : `Date.now() + Math.random()`. Le risque de collision est faible, mais la méthode n'est pas idéale.
- **Assignations orphelines** : si le JSON est modifié à la main, `assgn{}` peut pointer vers des éléments inexistants, sans erreur visible.
- **Aimant** : sur des points très denses, un groupe peut couvrir un repère voisin. Il faut alors baisser la distance de l'aimant ou la taille des pictos.

---

## Notes pour la reprise avec Claude Code

- Tout le code est dans `plan-electrique.html` : CSS, HTML et JS inline.
- L'état vit dans l'objet global `S`. Il n'y a pas de réactivité : les panneaux se redessinent via `upPanels()` et le canevas via `render()`.
- En mode impression, `render()` délègue à `renderPrint()`, et les interactions d'édition sont désactivées (`if (S.printMode) return`).
- Coordonnées : les éléments sont en pixels de l'image du plan (monde). La conversion vers l'écran passe par `w2c()` / `c2w()`, selon `S.view`.
- Toute modification de l'état doit appeler `pushHistory()` avant, pour rester annulable.
- Fonctions clés : `render()`, `upPanels()`, `autoAssign()`, `renderCP()`, `renderBP()`, `linkGroup()`, `drawSheet()`, `drawChecklist()`.
- Pour tester : charger `test-3-circuits.json`.
