---
name: linkedin-post
description: Génère un post LinkedIn concis et expert à partir d'un article de blog Liksi. Prend un chemin de fichier en argument. Post publié par l'auteur de l'article.
---

# Génération de post LinkedIn

Ce skill lit un article de blog et génère un post LinkedIn court, direct, qui met l'expertise en avant sans slop.

## Workflow

1. Lire le fichier passé en argument (chemin absolu ou relatif).
2. Extraire depuis le front matter : `title`, `authors`, `tags`, `description`.
3. Lire l'introduction et les sections principales pour identifier le **vrai apport technique** ou la **surprise** de l'article.
4. Générer 2 à 3 variantes de post LinkedIn.
5. Passer chaque variante par le skill `/deslopify` pour supprimer les tournures artificielles avant de les proposer.
6. Proposer les variantes deslopifiées à l'utilisateur pour qu'il choisisse.

## Format du post

Structure à respecter (inspirée de l'exemple de référence) :

```
[Hook : 1 phrase percutante — la surprise, l'insight ou le résultat concret]

[1-2 phrases max : ce qu'on a fait, ce que ça débloque, sans généralités]

➡️ [URL — laisser un placeholder si inconnue]

#Tag1 #Tag2 #Tag3 #Tag4 #Liksi
```

**Règles strictes :**
- Pas de liste à puces
- Pas d'intro du type "Ravi de partager", "Super article", "On a exploré"
- Pas d'emoji sauf 1 en accroche si pertinent (⚡️, 🔍, etc.) et la flèche ➡️ pour le lien
- Le hook doit nommer la techno ou le résultat précis, pas une généralité
- Hashtags : 3 à 5, basés sur les tags de l'article + `#Liksi`
- Ton : celui du praticien qui partage quelque chose qui a marché, pas du commercial

## Règles de ton

- Direct, pas enthousiaste de façon artificielle
- L'auteur parle en son nom, à la première personne ou "on" (collectif Liksi)
- Ce qui compte : ce que le lecteur va apprendre ou retenir, pas ce que l'article "couvre"
- Éviter les verbes creux : explorer, aborder, plonger, partager

## Exemple de référence

```
Transformer une API Spring en serveur MCP : c'est (vraiment) possible en 2 annotations. ⚡️

On détaille notre retour d'expérience et la mise en place technique sur le blog LIKSI.

➡️ https://lnkd.in/ezdx49za

#SpringAI #MCP #Kotlin #SpringBoot #Liksi
```
