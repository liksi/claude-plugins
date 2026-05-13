---
name: critique-content
description: Critique structurée et honnête d'un contenu tech/conseil (slides, article, doc). Points forts, faiblesses, angles morts, recommandations. Ton de pair exigeant, pas de complaisance.
---

# Critique de contenu tech/conseil

Ce skill lit un contenu (slides, article, document) et produit une analyse critique structurée : ce qui tient la route, ce qui est contestable, ce qui manque.

## Workflow

1. Déterminer la source du contenu à partir de l'argument :
   - Si l'argument ressemble à un chemin de fichier (commence par `/`, `./`, `~/`, ou se termine par une extension connue comme `.md`, `.txt`, `.pdf`, etc.), lire le fichier avec le tool Read.
   - Sinon, traiter l'argument directement comme le contenu à critiquer.
2. Lire le guide des tropes IA dans [`../deslopify/references/style_guide.md`](../deslopify/references/style_guide.md).
3. Identifier le type de contenu, l'audience visée, la thèse centrale.
4. Évaluer selon les 6 axes ci-dessous.
5. Produire le rapport critique dans le format défini.

## 5 axes d'évaluation

### 1. Solidité des arguments
- Les claims sont-ils étayés par des faits, des données, des exemples concrets ?
- Les sources sont-elles citées, récentes, fiables ? Y en a-t-il là où il le faudrait ?
- Y a-t-il des généralisations abusives, des corrélations présentées comme des causalités ?

### 2. Cohérence narrative
- Y a-t-il un fil conducteur clair du début à la fin ?
- Le problème posé reçoit-il une réponse satisfaisante ?
- Les transitions entre sections sont-elles logiques ou abruptes ?

### 3. Adéquation au public
- Le niveau de technicité est-il calibré à l'audience supposée ?
- Les exemples sont-ils parlants pour cette audience ou trop abstraits ?
- Y a-t-il des concepts supposés acquis qui mériteraient une explication ?

### 4. Angles morts
- Quels contre-arguments importants ne sont pas traités ?
- Quelles perspectives manquent (ex : opérationnel, sécurité, RH, coûts cachés, risques) ?
- Y a-t-il des simplifications qui pourraient fragiliser la crédibilité auprès d'un expert ?

### 5. Actionabilité
- Les recommandations sont-elles concrètes et réalisables ?
- Le lecteur/spectateur sait-il quoi faire après avoir consommé ce contenu ?
- Les next steps sont-ils priorisés ou tout est-il au même niveau d'urgence ?

### 6. Slop et tropes IA
En s'appuyant sur le guide [`../deslopify/references/style_guide.md`](../deslopify/references/style_guide.md), repérer les patterns caractéristiques d'un texte généré ou fortement assisté par IA :
- Parallélisme négatif ("Ce n'est pas X, c'est Y"), fragments courts dramatiques, questions rhétoriques auto-répondues
- Adverbes magiques (fondamentalement, objectivement, remarquablement), verbes pompeux (servir de, représenter)
- Structures en tricolon, anaphores en série, listes déguisées en prose
- Fausse vulnérabilité, inflation des enjeux, conclusions signalées ("En résumé", "En conclusion")
- Em-dashes compulsifs, puces toutes en gras, flèches unicode
- One-point dilution : une seule idée rephrasée dix fois

Ne signaler que les occurrences réelles présentes dans le texte, pas une liste générique.

## Format du rapport

Produire la critique en français, directe et sans complaisance. Ton : celui d'un pair exigeant qui veut que le contenu soit meilleur, pas d'un commercial qui ménage les susceptibilités.

Structure à respecter :

**Thèse centrale** (1 phrase : ce que le contenu cherche à démontrer)

**Points forts** (2 à 4 items max, avec justification courte)

**Points faibles** (2 à 4 items, avec justification — dire pourquoi ça pose problème, pas juste le constater)

**Angles morts** (1 à 3 sujets critiques absents ou sous-traités)

**Slop détecté** (tropes IA présents dans le texte — lister uniquement les occurrences réelles avec citation courte. Si aucun trope significatif, écrire "Aucun trope notable détecté.")

**Verdict global** (1 à 2 phrases brutalement honnêtes)

**3 recommandations priorisées** (concrètes, actionnables, ordonnées par impact)

## Règles de ton

- Pas de "c'est très bien mais..." — aller droit au but
- Pas de formules de politesse en ouverture ou fermeture
- Nommer les problèmes avec précision, pas avec des euphémismes
- Si quelque chose est franchement faible, le dire clairement
- Si quelque chose est fort, expliquer pourquoi (pas juste "bon point")
