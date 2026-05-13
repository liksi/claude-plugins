# AGENT.md — liksi-claude-plugins

Marketplace privé de plugins Claude Code pour les équipes Liksi.

## Structure du repo

```
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json          # Métadonnées : name, description, version
    skills/
      <skill-name>/
        SKILL.md           # Définition du skill (frontmatter + instructions)
        references/        # Fichiers de référence optionnels (ex: style_guide.md)
README.md                  # Liste des plugins disponibles
AGENT.md                   # Ce fichier
```

## Conventions

- Un plugin = un dossier dans `plugins/`
- Le nom du dossier = le nom du plugin = le nom du skill principal
- `SKILL.md` contient un frontmatter YAML avec `name` et `description`
- Les skills sont installés localement depuis `~/.claude/skills/<skill-name>/SKILL.md`

## Plugins existants

| Plugin | Skill principal | Rôle |
|--------|----------------|------|
| `critique-content` | `critique-content` | Critique structurée d'un contenu tech (slides, article, doc) |
| `deslopify` | `deslopify` | Supprime les tropes IA d'un texte |
| `linkedin-post` | `linkedin-post` | Génère un post LinkedIn à partir d'un article de blog Liksi |

## Ajouter un plugin

1. Créer `plugins/<nom>/.claude-plugin/plugin.json`
2. Créer `plugins/<nom>/skills/<nom>/SKILL.md`
3. Mettre à jour `README.md` (tableau des plugins)
4. Mettre à jour ce fichier (tableau des plugins)

## Déploiement local d'un skill

Copier le `SKILL.md` dans `~/.claude/skills/<nom>/SKILL.md` pour l'activer dans Claude Code.
