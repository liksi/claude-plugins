# Code Review

Effectue une revue de code orientée Software Craftsmanship sur les fichiers modifiés ou sur les fichiers fournis.

## Ce que la revue couvre

- Lisibilité et nommage
- Respect du principe de responsabilité unique
- Duplication et opportunités d'abstraction
- Gestion des erreurs et cas limites
- Couverture de test (existence et pertinence)
- Complexité cyclomatique et profondeur d'imbrication

## Usage

```
/code-review
/code-review path/to/file.ts
/code-review --focus=tests
```

## Comportement

La revue est formulée comme un retour de pair : direct, sans complaisance, mais constructif. Elle signale les problèmes avec leur niveau de sévérité (bloquant / conseil / nitpick) et propose des pistes concrètes plutôt que de simples constats.
