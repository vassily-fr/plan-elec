# PlanÉlec — notes pour Claude

Outil de plan électrique résidentiel, **un seul fichier** : `plan-electrique.html` (CSS, HTML et JS vanilla inline, aucune dépendance sauf Google Fonts). Publié sur GitHub Pages : https://vassily-fr.github.io/plan-elec/plan-electrique.html. Le README décrit les fonctionnalités, l'état `S{}`, les types `T{}` et les normes NF C 15-100 : le lire avant un gros changement.

## Conventions de code

- Style compact existant : une logique par ligne longue, noms courts (`S`, `T`, `lvl()`, `pc()`, `psym`). S'y tenir, pas de reformatage ni de découpage en modules.
- Interface, commentaires et textes imprimés **en français** ; messages de commit en anglais.
- Les sections du script sont repérées par `// ═══════ NOM ═══════` : ranger le nouveau code dans la bonne.
- Toute modification de l'état : `pushHistory()` **avant** (annulable, et déclenche la sauvegarde auto via `markDirty()`), puis `render()` et/ou `upPanels()`.
- Coordonnées « monde » = pixels de l'image du plan ; écran via `w2c()` / `c2w()`.
- Les ids de circuit/élément sont des nombres : comparer avec `String(a)===String(b)` comme le reste du code.
- Nouveau type d'élément : l'ajouter à `T{}`, à la barre gauche (`.eb[data-type]`), à `drawEl` (écran), `psym` (impression), `REPP` (repère), `DEFH` (hauteur), `naStep` (check-list) si besoin, et au tableau du README.

## Impression

- `drawSheet()` (plan) et `drawChecklist()` (check-list) dessinent sur un canvas en mm × `k` ; options dans `PDEF` / `S.cfg.print`, lues via `pc()`.
- Couleurs de circuit : `CC` à l'écran, `PCC` (mêmes teintes, assombries) sur papier via `pcol(i)` ; au-delà de 8 circuits, `pdash()` met les gaines en pointillé. Garder les deux palettes alignées par indice.
- Sans l'option « Couleurs », l'impression doit rester entièrement noir et blanc.

## Vérifier un changement

- Ouvrir le fichier directement (`file://…/plan-electrique.html`) dans le navigateur intégré ; `python3 -m http.server` (`.claude/launch.json`) échoue dans le sandbox.
- Le navigateur garde le projet de l'utilisateur en sauvegarde auto (IndexedDB) : **ne pas appeler `loadData()` avec des données de test**. Pour l'impression, rendre hors écran : sauvegarder `S.circs`/`S.assgn`, injecter les données de `test-3-circuits.json`, appeler `drawSheet`/`drawChecklist` sur un canvas détaché, restaurer, afficher l'image dans un calque temporaire puis le retirer.

## Documentation

- Chaque fonctionnalité visible met à jour le README (section concernée, et « Roadmap » / « Bugs connus » si utile).
- Format JSON : `projectData()` écrit `v:5` ; `loadData()` accepte v3 à v5 (migration v3 des gaines). Un nouveau champ doit être optionnel à la lecture (`d.x||défaut`) ; si le format change vraiment, monter `v`, ajouter la migration et garder les anciennes versions acceptées.
