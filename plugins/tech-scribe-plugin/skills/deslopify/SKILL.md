---
name: deslopify
description: Make text more genuine, natural, and feel not written by an AI or LLM by removing AI tropes and cliches. Use when asked to deslopify, naturalize, or remove AI tropes from text (FR : « deslopifier », « rendre plus naturel », « enlever le côté IA »).
---
# Deslopify Text

Rewrite or edit text so that it sounds human and genuine, free of the repetitive, patronizing, or grandiose patterns typical of AI-generated content.

## Workflow

1. Read the text provided by the user.
2. Read the AI writing tropes list in [references/style_guide.md](references/style_guide.md) before editing: it defines the anti-patterns to remove.
3. If the text is in French, also read [references/style_guide_fr.md](references/style_guide_fr.md): French-specific tics, and where they override `style_guide.md` (French « » quotes, sentence length, fixing repetition).
4. Identify the tropes actually present in the text (e.g., magic adverbs, the "delve" family, negative parallelism, false vulnerability, listicles in a trench coat).
5. Rewrite the text to eliminate them while preserving its meaning, intent, and language. Prefer clear, direct, varied prose; don't swap one trope for another.

## User preferences

- **Pas d'em dashes (—)** : remplacer systématiquement les em dashes par une alternative naturelle selon le contexte : virgule, deux-points, parenthèses, ou reformulation de la phrase.
