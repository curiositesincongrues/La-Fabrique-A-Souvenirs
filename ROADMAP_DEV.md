# 🗺 Roadmap de Développement — La Fabrique à Souvenirs
## Du Sanctuaire des 9 Nekos au moteur générique

> *Construire progressivement, valider à chaque étape, ne jamais avancer sur du sable.*

---

## 🧭 Stratégie générale

**Nouveau repo** `la-fabrique-a-souvenirs` — pas un fork, une extraction chirurgicale.
**Le Sanctuaire des 9 Nekos** devient le premier projet exemple chargeable dans la Fabrique.
**Règle d'or** : chaque bloc produit quelque chose de jouable ou de testable. On ne code pas dans le vide.

---

## 📦 Ce qu'on extrait du Sanctuaire (copie directe)

Avant d'écrire une ligne, on copie ces fichiers dans le nouveau repo :

| Fichier source | Destination Fabrique | Pourquoi |
|---|---|---|
| `js/audio.js` | `engine/audio.js` | speakDucked, ducking, Web Audio — déjà propre |
| `js/scan.js` | `runtime/triggers/scan_qr.js` | QR scan natif — déjà fonctionnel |
| `sw.js` | `sw.js` | PWA base — copie quasi telle quelle |
| `manifest.json` | `manifest.json` | PWA manifest — adapter nom/icônes |
| `qr.html` | `studio/qr-generator.html` | Génération QR — extraire la logique |
| `texts.json` (pattern) | `data/project.schema.json` | Le pattern de centralisation des textes |
| `css/` (variables CSS) | `css/tokens.css` | Les custom properties de base |

**Ce qu'on ne porte pas** : logique des 9 gardiens, hub hardcodé, WebGL mood, cinématique spécifique.

---

## 🧱 BLOC 0 — Fondations (3 jours)
*Objectif : nouveau repo propre, structure posée, rien ne casse.*

### Tâches
- [ ] Créer repo `la-fabrique-a-souvenirs` sur GitHub
- [ ] Structure de dossiers :
```
/
├── index.html          ← point d'entrée Studio
├── runtime.html        ← point d'entrée Runtime (joueur)
├── manifest.json
├── sw.js
├── css/
│   ├── tokens.css      ← variables globales (extrait du Sanctuaire)
│   ├── studio.css
│   └── runtime.css
├── js/
│   ├── engine/
│   │   ├── audio.js    ← copié du Sanctuaire
│   │   └── project.js  ← nouveau : lecture/écriture JSON projet
│   ├── studio/
│   │   └── studio.js   ← nouveau : point d'entrée Studio
│   ├── runtime/
│   │   ├── runtime.js  ← nouveau : point d'entrée Runtime
│   │   └── triggers/
│   │       └── scan_qr.js  ← copié du Sanctuaire
├── data/
│   ├── project.schema.json   ← schéma de données figé
│   └── example_nekos.json    ← Le Sanctuaire des 9 Nekos converti
└── assets/
    └── audio/          ← sons CC0
```
- [ ] GitHub Pages activé sur `main`
- [ ] `sw.js` + `manifest.json` adaptés (nom, icônes)
- [ ] `tokens.css` extrait et nettoyé

### ✅ Validation
Ouvrir `runtime.html` sur mobile → la page se charge, le SW s'enregistre, aucune erreur console.

---

## 🧱 BLOC 1 — Schéma de données + Runtime minimal (5 jours)
*Objectif : charger un fichier JSON et afficher une étape. Rien de plus.*

### Tâches
- [ ] Figer `project.schema.json` (les modèles du README — ne plus les modifier après ce bloc)
- [ ] Convertir Le Sanctuaire des 9 Nekos en `example_nekos.json` (première instance du nouveau schéma)
- [ ] `project.js` : fonctions `loadProject(json)`, `getStep(id)`, `getActor(id)`, `getAsset(id)`
- [ ] `runtime.js` : charger un projet depuis URL param `?project=example_nekos`
- [ ] Afficher `step.titre` + `step.intro` + `step.images.on_arrive`
- [ ] Afficher le personnage (`actor`) en overlay avec sa `phrase_accueil`
- [ ] Navigation manuelle vers `step.nextStep` (bouton "Continuer" — pas encore de trigger)

### ✅ Validation
`runtime.html?project=example_nekos` → l'intro de l'étape 1 du Sanctuaire s'affiche avec l'image et le personnage. Le bouton "Continuer" passe à l'étape 2.

---

## 🧱 BLOC 2 — Triggers QR + Code (5 jours)
*Objectif : les deux triggers MVP fonctionnent dans le Runtime.*

### Tâches
- [ ] `triggers/scan_qr.js` : adapter `js/scan.js` du Sanctuaire à l'interface trigger générique
  ```js
  // Interface unifiée pour tous les triggers
  trigger.activate(triggerData) → Promise<{success, message}>
  ```
- [ ] `triggers/enter_code.js` : input texte, comparaison `case_sensitive`, `max_attempts`
- [ ] Logique d'indices progressifs : `hints[0]` → `hints[1]` → `hints[2]` selon tentatives
- [ ] Affichage `step.images.on_success` après trigger réussi
- [ ] Affichage `step.images.on_reward` + passage à `nextStep`
- [ ] Fallback : si `scan_qr` échoue 3 fois → proposer `enter_code`
- [ ] Mettre à jour `example_nekos.json` avec de vrais triggers QR/code

### ✅ Validation
Scanner un QR du Sanctuaire imprimé → l'étape se déverrouille. Taper le mauvais code 2 fois → indice 2 apparaît. Taper le bon code → image de succès, puis image de récompense.

---

## 🧱 BLOC 3 — Audio Engine (4 jours)
*Objectif : son ambiant + TTS + ducking fonctionnent. Aucune régression mobile.*

### Tâches
- [ ] Adapter `engine/audio.js` du Sanctuaire :
  - Remplacer les références hardcodées par des paramètres `(soundId, config)`
  - `playAmbient(soundId, fadeMs)` — crossfade entre étapes
  - `stopAmbient(fadeMs)`
  - `speakDucked(text, voiceConfig)` — TTS avec ducking automatique
  - `playSuccess(assetId)` — son de succès (fichier ou enregistrement)
- [ ] Bibliothèque d'ambiances CC0 : 6 fichiers audio embarqués dans `/assets/audio/`
- [ ] `tts_config` lu depuis `project.audio` (voix, pitch, rate)
- [ ] Ducking automatique : GainNode sur la soundtrack pendant le TTS
- [ ] Test mobile iOS (déblocage audio sur geste utilisateur)

### ✅ Validation
Arriver sur une étape → ambiance démarre en fondu. Le personnage parle → musique descend. Fin du TTS → musique remonte. Passer à l'étape suivante → crossfade propre.

---

## 🧱 BLOC 4 — Soundtrack Continue + Mastering (3 jours)
*Objectif : la musique narrative traverse tout le jeu avec sa propre progression.*

### Tâches
- [ ] 4 arcs musicaux CC0 embarqués (mystere, aventure, melancolie, triomphe)
- [ ] `engine/soundtrack.js` : calcule la phase (intro/milieu/climax/résolution) selon `currentStepIndex / totalSteps`
- [ ] Transition automatique entre phases : crossfade 2s
- [ ] Montée progressive vers le climax à l'avant-dernière étape
- [ ] `audio_mix` lu depuis `project.audio.mix` : volumes relatifs, timings
- [ ] En mode avancé : sliders de mix exposés dans le Studio

### ✅ Validation
Jouer les 3 étapes de test → la musique progresse de l'intro vers la résolution. À l'avant-dernière étape, le climax s'installe. À la dernière, la résolution.

---

## 🧱 BLOC 5 — Journal + Score Poésie + Estampe (5 jours)
*Objectif : le joueur reçoit un souvenir tangible à la fin.*

### Tâches
- [ ] `runtime/journal.js` : enregistre en `sessionStorage` chaque étape franchie (timestamp, tentatives, indices)
- [ ] Calcul du `score_poesie` : formule basée sur tentatives, indices, temps par étape
- [ ] `runtime/cinematic.js` :
  - Défilement des personnages avec leur phrase (TTS + illustration)
  - Affichage du `closing_word` lettre par lettre
  - Fondu blanc
- [ ] `runtime/selfie.js` : `getUserMedia()`, capture, canvas overlay
- [ ] `runtime/estampe.js` : Canvas API, composition PNG (titre + selfie + score + closing_word + illustration)
- [ ] Bouton export PNG + partage natif (`navigator.share()` si dispo)

### ✅ Validation
Finir l'exemple Nekos → cinématique joue 25 secondes → module selfie s'ouvre → estampe générée avec photo → bouton "Télécharger" fonctionnel sur mobile.

---

## 🧱 BLOC 6 — Branchements Narratifs (4 jours)
*Objectif : une étape peut mener à deux chemins différents.*

### Tâches
- [ ] `triggers/branch.js` : affiche deux boutons de choix, enregistre la décision dans le journal
- [ ] `runtime.js` : gestion de `step.branches.enabled` → bifurque vers `options[n].nextStep`
- [ ] Mise à jour `example_nekos.json` avec un branchement exemple
- [ ] Le journal enregistre `branche_choisie` pour chaque étape
- [ ] L'estampe mentionne le chemin emprunté (symbole discret)

### ✅ Validation
Arriver à une étape avec branchement → deux boutons apparaissent → chaque choix mène à une étape différente → les deux chemins convergent vers la fin → l'estampe reflète le chemin.

---

## 🧱 BLOC 7 — Thèmes Visuels (4 jours)
*Objectif : 5 skins complets, application automatique au chargement du projet.*

### Tâches
- [ ] `css/skins/` : 5 fichiers CSS (variables + overrides)
  - `foret_enchantee.css`, `ville_mystere.css`, `espace.css`, `fond_marin.css`, `neutre.css`
- [ ] `runtime.js` : lit `project.visual_skin`, injecte le skin au chargement
- [ ] Chaque skin : fond, typographie, boutons, overlays, icônes trigger, transition spécifique
- [ ] Test sur tous les skins avec l'exemple Nekos

### ✅ Validation
`runtime.html?project=example_nekos&skin=ville_mystere` → tout le Runtime bascule en palette nuit/néon avec glitch subtil.

---

## 🧱 BLOC 8 — Studio : Projet + Étapes (7 jours)
*Objectif : créer un projet simple depuis l'interface Studio.*

### Tâches
- [ ] `studio/studio.js` : état central du projet en mémoire (`currentProject`)
- [ ] Écran "Mon Jeu" : titre, skin, arc musical, image de couverture
- [ ] Écran "Mes Étapes" : liste des étapes, réordonnement drag-and-drop
- [ ] Écran "Une Étape" : formulaire complet (intro, actor, images, trigger, hints, reward, branch)
- [ ] Sélecteur trigger en icônes uniquement
- [ ] Sauvegarde `localStorage` automatique à chaque modification
- [ ] Export JSON : bouton "Télécharger mon projet"
- [ ] Import JSON : bouton "Charger un projet" + drag-drop

### ✅ Validation
Créer un projet de 3 étapes depuis zéro dans le Studio → exporter le JSON → le charger dans `runtime.html` → jouer les 3 étapes jusqu'à l'estampe.

---

## 🧱 BLOC 9 — Studio : Personnages + Assets + Son (5 jours)
*Objectif : le créateur peut gérer ses personnages, images et sons.*

### Tâches
- [ ] Écran "Mes Personnages" : créer/éditer un actor (nom, illustration upload, voice_id, phrase_accueil)
- [ ] Écran "Mes Images" :
  - Upload local → stocké en `base64` dans le projet JSON
  - Bibliothèque CC0 intégrée (20 images libres)
  - Picker avec 3 onglets : `on_arrive` / `on_success` / `on_reward`
- [ ] Enregistrement son de succès : MediaRecorder API, 3 sec max, waveform visuelle
- [ ] Picker d'ambiance sonore : 6 options avec preview au hover

### ✅ Validation
Créer un personnage, l'assigner à une étape, uploader 3 images, enregistrer un son de succès → tout apparaît correctement dans le Runtime.

---

## 🧱 BLOC 10 — Studio : Preview + Mode Fantôme (4 jours)
*Objectif : le créateur peut tester sans quitter le Studio.*

### Tâches
- [ ] Preview rapide : panneau latéral glissant, Runtime embarqué en iframe
- [ ] Bouton "Passer ce trigger" semi-transparent dans la preview
- [ ] Preview plein écran : ouvre `runtime.html` avec le projet courant en `sessionStorage`
- [ ] Mode Fantôme : overlay translucide sur chaque étape avec les notes de conception
- [ ] Bouton de retour vers le Studio depuis la preview

### ✅ Validation
Créer un projet, lancer la preview plein écran, passer tous les triggers en un clic, activer le mode fantôme → les notes apparaissent en transparence sur chaque étape.

---

## 🧱 BLOC 11 — Génération QR + PDF Impression (5 jours)
*Objectif : le créateur peut préparer physiquement son aventure.*

### Tâches
- [ ] `engine/qr-generator.js` : adapter `qr.html` du Sanctuaire
  - Générer un QR par étape (data = `project_id + step_id`)
  - Animation SVG pulsante sur le QR (`qr_animated: true`)
- [ ] `engine/pdf-export.js` : `jsPDF` + `html2canvas`
  - Page de couverture (titre + illustration)
  - Une page par étape : QR + étiquette + indice de placement
  - Checklist créateur de mise en place
  - Fiche enveloppe à découper

### ✅ Validation
Cliquer "Préparer l'aventure" → PDF de 6 pages généré en moins de 3s → imprimable, QR codes fonctionnels.

---

## 🧱 BLOC 12 — Onboarding : Tutoriel Vivant (7 jours)
*Objectif : premier lancement = mini-aventure jouable avant de créer.*

### Tâches
- [ ] `data/tutoriel_atelier.json` : projet pré-rempli de 3 étapes avec guide actor
- [ ] Détection premier lancement (`localStorage.getItem('fabrique_onboarded')`)
- [ ] Si premier lancement : lancer `runtime.html?project=tutoriel_atelier&mode=onboarding`
- [ ] En mode `onboarding` : bulles d'explication superposées à chaque étape
- [ ] Écran de fin du tutoriel : *"Maintenant c'est toi le créateur"* → choix modifier / créer nouveau
- [ ] `studio/guide.js` : assistant contextuel (détection blocage par timers + champs vides)
- [ ] Table des 5 blocages avec questions contextuelles
- [ ] Fiche de préparation : écran dédié + version imprimable HTML

### ✅ Validation
Effacer le `localStorage` → ouvrir la Fabrique → le tutoriel se lance automatiquement → le jouer jusqu'à la fin → arriver sur le Studio avec le projet tutoriel chargé et modifiable.

---

## 🧱 BLOC 13 — Progression Créateur + Badges (4 jours)
*Objectif : la complexité se révèle progressivement.*

### Tâches
- [ ] `engine/progression.js` : suivi des badges en `localStorage`
- [ ] 6 badges implémentés avec leurs conditions de déclenchement
- [ ] Chaque badge débloque un élément UI (trigger GPS, branchements, mode duo...)
- [ ] Notification discrète à l'obtention d'un badge (pas de popup agressif)
- [ ] Vue "Mon parcours" dans le Studio : badges obtenus + prochaine étape

### ✅ Validation
Publier un premier projet → badge "Premier pas" → trigger `reach_location` apparaît dans le sélecteur. Vérifier que les autres badges restent verrouillés.

---

## 🧱 BLOC 14 — Triggers Avancés : GPS + Find + Action (7 jours)
*Objectif : les 3 triggers restants fonctionnent.*

### Tâches
- [ ] `triggers/reach_location.js` : `navigator.geolocation.watchPosition()`, rayon, confirmation, fallback
- [ ] `triggers/find_object.js` : description, bouton de confirmation
- [ ] `triggers/perform_action.js` : description, bouton, `uses_sensor` (optionnel : accéléromètre)
- [ ] Studio : formulaires de configuration pour ces 3 triggers (mode avancé)
- [ ] Test mobile Android + iOS pour la géoloc

### ✅ Validation
Créer une étape GPS pointant sur une vraie adresse → approcher à moins de 15m → l'étape se déverrouille. Tester le fallback code si hors zone.

---

## 🧱 BLOC 15 — Mode Duo (6 jours)
*Objectif : deux enfants créent ensemble.*

### Tâches
- [ ] `engine/duo-session.js` : génération d'un `session_id` unique, QR de partage
- [ ] Partage du projet via URL `?session=abc123` (projet stocké dans un GitHub Gist temporaire)
- [ ] Séparation des rôles dans le Studio : host (textes/hints/rewards) / guest (images/actors/sons)
- [ ] Merge à la sauvegarde : les modifications des deux créateurs se combinent
- [ ] En mode enfant : écran simplifié "Créer à deux" avec QR à montrer à l'ami

### ✅ Validation
Ouvrir la Fabrique sur deux appareils via le QR de session → host modifie l'intro d'une étape / guest change l'image → sauvegarder → les deux modifications apparaissent dans le projet final.

---

## 🧱 BLOC 16 — Partage, GitHub, Galerie (8 jours)
*Objectif : un projet peut être publié et partagé via lien.*

### Tâches
- [ ] `engine/github.js` : connexion GitHub via token (OAuth simplifié ou PAT)
- [ ] Publication : créer/mettre à jour un repo `fabrique-[titre]` sur le compte de l'utilisateur
- [ ] Déploiement GitHub Pages automatique
- [ ] Génération lien unique : `[user].github.io/fabrique-[titre]`
- [ ] Mode Rejeu créateur : overlay stats (tentatives, indices, abandons) via GitHub Issues API
- [ ] Galerie publique : `la-fabrique-galerie` repo central avec un JSON index
- [ ] Soumission galerie : Pull Request auto avec le JSON du projet

### ✅ Validation
Publier un projet → lien partageable fonctionnel → un ami sur un autre appareil joue via ce lien → les stats apparaissent dans le mode rejeu créateur le lendemain.

---

## 🧱 BLOC 17 — Bibliothèque de Fragments (4 jours)
*Objectif : les blocs créés se réutilisent entre projets.*

### Tâches
- [ ] `engine/fragments.js` : CRUD des fragments en `localStorage`
- [ ] Sauvegarde automatique en fragment lors de la création d'un actor / ambient / challenge
- [ ] Picker "Utiliser un fragment existant" dans les écrans Studio concernés
- [ ] Export/import de la bibliothèque en JSON
- [ ] Partage de fragments entre créateurs (via GitHub Gist)

### ✅ Validation
Créer un personnage dans le projet A → l'ouvrir en fragment → créer le projet B → utiliser le même personnage via le picker → les deux projets partagent la même définition d'actor.

---

## 🧱 BLOC 18 — Polissage & Production (5 jours)
*Objectif : tout fonctionne proprement sur mobile Android + iOS.*

### Tâches
- [ ] Audit complet mobile (touch, caméra, audio unlock, géoloc)
- [ ] Service Worker : stratégie de cache fine (runtime offline, studio online)
- [ ] Performance : lazy loading des assets audio, compression images base64
- [ ] Accessibilité : contrastes, tailles tactiles min 44px, labels ARIA
- [ ] `?debug=hard` porté depuis le Sanctuaire : overlay de test rapide
- [ ] README final du repo Fabrique (version technique du doc)

### ✅ Validation
Jouer un projet complet de 5 étapes sur mobile en mode avion (après premier chargement) → zéro erreur, son, images, QR scan, estampe → tout fonctionne.

---

## 📊 Vue d'ensemble

```
BLOC 0   Fondations                    ████ 3j
BLOC 1   Schéma + Runtime minimal      ████████ 5j
BLOC 2   Triggers QR + Code            ████████ 5j
BLOC 3   Audio Engine                  ██████ 4j
BLOC 4   Soundtrack + Mastering        █████ 3j
BLOC 5   Journal + Estampe + Selfie    ████████ 5j
BLOC 6   Branchements narratifs        ██████ 4j
BLOC 7   Thèmes visuels                ██████ 4j
────────────────────── MVP RUNTIME COMPLET ──────────────────────
BLOC 8   Studio : Projet + Étapes      ███████████ 7j
BLOC 9   Studio : Perso + Assets + Son ████████ 5j
BLOC 10  Studio : Preview + Fantôme    ██████ 4j
BLOC 11  QR Generator + PDF Impression ████████ 5j
────────────────────── MVP STUDIO COMPLET ───────────────────────
BLOC 12  Onboarding + Tutoriel vivant  ███████████ 7j
BLOC 13  Progression + Badges          ██████ 4j
────────────────────── PÉDAGOGIE COMPLÈTE ───────────────────────
BLOC 14  Triggers avancés (GPS...)     ███████████ 7j
BLOC 15  Mode Duo                      █████████ 6j
BLOC 16  Partage + GitHub + Galerie    ████████████ 8j
BLOC 17  Bibliothèque de Fragments     ██████ 4j
BLOC 18  Polissage & Production        ████████ 5j
────────────────────── VERSION COMPLÈTE ─────────────────────────

Total estimé : ~105 jours de développement solo itératif
MVP jouable (Blocs 0→7) : ~33 jours
MVP créateur (Blocs 8→11) : +21 jours = ~54 jours
```

---

## 🔑 Règles de progression

**Ne pas passer au bloc suivant tant que la validation du bloc courant n'est pas verte.**

Chaque bloc peut être développé sur une branche dédiée :
```
git checkout -b bloc-01-runtime-minimal
# ... développement ...
git checkout main && git merge bloc-01-runtime-minimal
```

**Commits atomiques.** Un commit = une fonctionnalité. Jamais de "misc fixes".

**Le Sanctuaire reste intact.** On ne modifie pas l'ancien repo — on extrait, on adapte, on améliore.

---

## 🎯 Premier sprint recommandé

Blocs 0, 1, 2 en séquence. Résultat en ~13 jours : le Runtime charge le Sanctuaire des 9 Nekos converti en JSON, les QR du vrai jeu fonctionnent, les indices progressifs s'affichent. C'est la preuve que l'architecture tient.

*À partir de là, la Fabrique existe vraiment.*
