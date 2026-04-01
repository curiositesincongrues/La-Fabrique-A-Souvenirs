# 🤝 Contribuer à La Fabrique à Souvenirs

> *Ce projet est ouvert. Le code n'est pas une clôture — c'est une invitation.*

---

## Comment contribuer

### Signaler un bug
Ouvrir une [issue](../../issues/new?template=bug_report.md) avec le template prévu. Plus c'est précis (navigateur, appareil, étapes pour reproduire), plus vite c'est corrigé.

### Proposer une amélioration
Ouvrir une issue avec le label `enhancement` avant de coder quoi que ce soit. On discute d'abord — ça évite les PR qui ne peuvent pas être mergées.

### Soumettre une Pull Request

1. Forker le repo
2. Créer une branche nommée selon le bloc concerné :
   ```
   git checkout -b bloc-03-audio-engine
   ```
3. Commits atomiques — un commit = une fonctionnalité :
   ```
   feat: add crossfade between ambient sounds
   fix: audio unlock on iOS first gesture
   docs: update audio engine schema
   ```
4. Ouvrir la PR vers `main` avec une description claire

### Soumettre une aventure à la galerie
Utiliser le [template dédié](../../issues/new?template=nouvelle_aventure.md). L'aventure doit être publiée et jouable avant soumission.

---

## Conventions

**Langue** : code en anglais (variables, fonctions, commentaires), interface et contenu en français.

**Pas de bundler, pas de framework** — vanilla JS modulaire uniquement. Toute dépendance externe doit être justifiée et légère.

**Compatibilité mobile first** — tester sur Android Chrome et iOS Safari avant toute PR.

**Zéro donnée personnelle** — aucune analytics, aucun tracking, aucune donnée envoyée à un serveur tiers.

---

## Code de conduite

Ce projet accueille des contributions de tout le monde. Voir [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md).

---

*Fait avec cœur et rigueur, pour que le numérique redevienne un terrain de jeu poétique.*
