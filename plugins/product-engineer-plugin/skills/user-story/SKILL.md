---
name: user-story
description: Rédige une User Story fonctionnelle (Story / Périmètre fonctionnel / Critères d'acceptation / Jeux de recette / Points à clarifier / Référence technique). Utilise systématiquement ce skill dès que l'utilisateur demande de rédiger, créer, ajouter, compléter ou corriger une US, une story ou une spécification fonctionnelle, même sans mentionner explicitement le format.
---

# Skill — User Story fonctionnelle

## Objectif
Produire une User Story **fonctionnelle**, en français, exploitable directement par un consommateur métier (DATA, CRM, achats, compta, etc.), à partir d'une demande utilisateur et d'une source de référence (Confluence, spec, ticket).

Une US décrit ce que voit ou manipule l'utilisateur en **langage métier**. Ce n'est ni un cahier des charges, ni une spec technique. Si elle documente un existant, elle ne le re-spécifie pas.

## Quand déclencher
- L'utilisateur demande de rédiger / créer / compléter / corriger une US.
- L'utilisateur évoque un nouveau lot, un nouveau champ, un nouveau besoin à spécifier.
- L'utilisateur dit "fais une story pour…", "spec fonctionnelle de…", "documente le besoin de…".

## Quand ne PAS l'utiliser
- **Tâche purement technique** (refactor, dette technique, migration) : pas de valeur utilisateur → utiliser un ticket d'engineering, pas une US.
- **Story trop large** (plusieurs personas, plusieurs "Je veux", plusieurs parcours) : commencer par découper avant de rédiger.
- **Problème mal compris** : si le besoin métier n'est pas clair, écrire d'abord un énoncé de problème ou faire de la discovery — ne pas figer une US sur du vide.

## Format obligatoire

```markdown
# US-<identifiant> — <titre court orienté valeur métier>

## Story
**En tant que** <rôle/persona métier>
**Je veux** <action/besoin exprimé en langage métier>
**Afin de** <bénéfice attendu>

## Périmètre fonctionnel
<Ce que l'utilisateur voit ou manipule, en langage métier.>
- liste à puces des éléments exposés (libellés métier)
- préciser pagination/filtres si pertinent
- renvoyer vers une autre US par lien relatif si dépendance

## Critères d'acceptation
- **CA1** : <règle métier vérifiable>
- **CA2** : …
- **CA3** : …

## Jeux de recette  *(optionnel)*
<Identifiants ou exemples concrets fournis dans la source pour vérifier le comportement.>

## Points à clarifier  *(optionnel mais important)*
- <toute ambiguïté, champ marqué "Manquant"/"KO", règle non tranchée>

## Référence technique
<Lien vers la source de référence (Confluence, spec, ticket) qui porte le détail technique.>
```

> **Identifiant** : adapter à la convention du projet courant. Exemples : `US-V1-01`, `US-CRM-12`, `US-1234`. Si le projet a une convention (lots, epics, préfixes), la respecter.

## Règles de rédaction

### À faire
- **Langage métier uniquement** (rôle, action, bénéfice formulés du point de vue de l'utilisateur).
- **Concision** : phrases courtes, puces, pas de paragraphes longs.
- **Critères vérifiables** : un CA doit pouvoir être prouvé vrai ou faux.
- **Remonter tout flou** dans "Points à clarifier", surtout les champs marqués "Manquant"/"KO"/"TBD" dans la source.
- **Numéroter** les CA (`CA1`, `CA2`, …).
- **Lien relatif** vers les US dépendantes (`[US-XXX](US-XXX-slug.md)`).
- **Convention de nommage** des fichiers : suivre celle du projet (par défaut `US-<identifiant>-<slug>.md`).

### À ne pas faire
- ❌ Pas de noms de DTO, de classes, d'enums techniques dans le corps de l'US — les libellés métier équivalents les remplacent.
- ❌ Pas de mapping source/colonne, pas de SQL, pas de JSON, pas de noms de tables/vues.
- ❌ Pas d'inventer de champs ou de règles non présents dans la source.
- ❌ Pas plusieurs "Je veux" dans une même Story — si besoin, scinder en deux US.
- ❌ Pas de critère type "le système est plus rapide" / "les données sont fiables" (non vérifiable).
- ❌ Pas de détail d'implémentation côté pipeline ("l'import remonte une erreur", "le job recalcule…") — décrire l'effet observable côté utilisateur.

## Workflow recommandé

### 1. Lire la source
Confluence, spec, ticket, ou portion fournie par l'utilisateur.

### 2. Identifier le persona consommateur
Qui utilise réellement la donnée ou la fonctionnalité ?

> **Quality check** — Est-ce un persona précis ("utilisateur en essai", "commercial portefeuille") ou générique ("utilisateur") ? Préférer toujours le précis.

### 3. Rédiger la Story (En tant que / Je veux / Afin de)

> **Quality checks**
> - **"En tant que"** : persona précis, pas "utilisateur".
> - **"Je veux"** : une action métier de l'utilisateur, pas une fonctionnalité technique.
> - **"Afin de"** : motivation réelle, pas une reformulation de "Je veux".

### 4. Lister le périmètre fonctionnel
Éléments exposés côté utilisateur, en libellés métier. Pagination, filtres, dépendances vers d'autres US.

### 5. Rédiger les critères d'acceptation

> **Quality checks**
> - Chaque CA est **vérifiable** (prouvable vrai ou faux).
> - **Un seul "Quand"** et **un seul "Alors"** par scénario : si tu en as plusieurs, c'est probablement plusieurs US à découper.
> - Les "Étant donné" peuvent s'empiler — c'est normal.
> - **Alignement** : le "Quand" reflète le "Je veux", le "Alors" reflète le "Afin de".

### 6. Remonter les flous
Points à clarifier : champs "Manquant"/"KO"/"TBD", règles non tranchées, cas non observés.

### 7. Ajouter le lien vers la source
En référence technique — la source de vérité technique vit ailleurs (Confluence, Notion…).

### 8. Écrire le fichier
Selon la convention du projet (emplacement, nommage).

### 9. Valider
- **Relire à voix haute** : qui, quoi, pourquoi sont-ils évidents ?
- **Test QA mental** : peut-on écrire des cas de test à partir de chaque CA ?
- **Critère de taille** : si la story semble trop grosse → la découper.
- **INVEST** : Independent, Negotiable, Valuable, Estimable, Small, Testable.

## Adaptation au projet
Avant de rédiger, repérer dans le projet courant (CLAUDE.md, README, fichiers voisins) :
- la **convention d'identifiant** des US (préfixe, numérotation, lots/epics) ;
- l'**emplacement** des US (dossier `lots/`, `specs/`, `docs/us/`, etc.) ;
- la **source de référence** à pointer en "Référence technique" (Confluence, Notion, lien interne) ;
- les **personas métier** récurrents du projet.

Si ces conventions ne sont pas claires, demander avant de produire le fichier.

## Exemple minimal

```markdown
# US-V1-01 — Consultation des opérations facturées

## Story
**En tant que** consommateur DATA
**Je veux** consulter la liste des opérations commerciales avec leur commercial propriétaire, leur annonceur et leurs montants de CA
**Afin d'**alimenter le SI décisionnel avec les données de facturation

## Périmètre fonctionnel
Pour chaque opération exposée, l'utilisateur dispose de :
- l'**identifiant** de l'opération
- le **commercial propriétaire**
- l'**annonceur**
- le **CA à facturer** et le **CA facturé**
- la **date de dernière mise à jour**

La liste est **paginée** et **filtrable** par identifiant, commercial, annonceur, numéro de facture, période de facturation.

## Critères d'acceptation
- **CA1** : un consommateur peut récupérer une opération précise via son identifiant.
- **CA2** : les opérations en statut `Devis`, `Hypothèse` ou `Abandonné` ne contribuent pas aux montants CA.
- **CA3** : les montants CA sont figés au moment de l'import (pas de recalcul à la volée).

## Points à clarifier
- Calcul du CA à facturer pour une opération partiellement facturée : reliquat HT non facturé ?

## Référence technique
Spécification technique détaillée : <lien vers la source>
```

## Exemple — style Given/When/Then

Variante avec critères d'acceptation rédigés en scénarios (Gherkin) plutôt qu'en règles. À utiliser quand le besoin se prête mieux à un parcours utilisateur qu'à des règles déclaratives.

```markdown
# US-042 — Connexion Google pour les utilisateurs en essai

## Story
**En tant qu'**utilisateur en période d'essai visitant l'application pour la première fois
**Je veux** me connecter avec mon compte Google
**Afin de** pouvoir accéder à l'application sans créer ni mémoriser un nouveau mot de passe

## Périmètre fonctionnel
- bouton **"Se connecter avec Google"** visible sur la page de connexion
- redirection vers le parcours d'**onboarding** après authentification réussie
- objectif : réduire la friction à l'inscription pour les utilisateurs en essai

## Critères d'acceptation

**Scénario : un utilisateur en essai se connecte pour la première fois via Google OAuth**
- **Étant donné** que je suis sur la page de connexion
- **Et** que je possède un compte Google
- **Et** que le bouton "Se connecter avec Google" est visible
- **Quand** je clique sur "Se connecter avec Google" et que j'autorise l'application
- **Alors** je suis connecté à l'application
- **Et** je suis redirigé vers le parcours d'onboarding

## Référence technique
<lien vers la source>
```

## Anti-patterns fréquents

Format : **Symptôme → Conséquence → Correction**.

### ❌ Story technique déguisée
- **Symptôme** : "En tant que développeur, je veux refactorer l'API, afin d'avoir un code plus propre."
- **Conséquence** : aucune valeur utilisateur, c'est une tâche d'engineering — pas une US.
- **Correction** : si aucun consommateur métier n'est identifiable, créer un ticket de dette technique, pas une US.

### ❌ "En tant qu'utilisateur" (trop générique)
- **Symptôme** : toutes les US commencent par "En tant qu'utilisateur".
- **Conséquence** : pas de clarté sur le persona — des utilisateurs différents ont des besoins différents.
- **Correction** : utiliser un persona précis ("utilisateur en essai", "commercial portefeuille", "consommateur DATA", "administrateur").

### ❌ "Afin de" qui répète le "Je veux"
- **Symptôme** : "Je veux cliquer sur Enregistrer, afin de sauvegarder mon travail."
- **Conséquence** : aucune information sur la motivation réelle.
- **Correction** : creuser le pourquoi — "afin de ne pas perdre ma progression en cas de crash".

### ❌ Plusieurs Quand / Alors dans un même scénario
- **Symptôme** : un scénario contient 3 "Quand" et 4 "Alors".
- **Conséquence** : la story est trop large — plusieurs fonctionnalités empaquetées ensemble.
- **Correction** : découper en plusieurs US (une par paire Quand/Alors).

### ❌ Critère non vérifiable
- **Symptôme** : "CA1 : les données sont fiables." / "Alors l'utilisateur a une meilleure expérience."
- **Conséquence** : impossible à valider en recette.
- **Correction** : rendre mesurable — "les montants exposés correspondent aux montants de la source (pas de recalcul)".

### ❌ Détail technique dans le corps
- **Symptôme** : "Le champ `montantHt` est mappé sur `vue_facturation.MT_HT`."
- **Conséquence** : pollue la lecture métier, et le mapping va vite être obsolète.
- **Correction** : laisser le détail dans la source technique, garder "**montant HT**" en libellé métier.

### ❌ Enum technique exposé
- **Symptôme** : "Type : `ANNULE` | `SIGNE` | `A_FACTURER` | `FACTURE`."
- **Conséquence** : on lit du code, pas du métier.
- **Correction** : reformuler en libellés métier — "annulée / signée / à facturer / facturée".
