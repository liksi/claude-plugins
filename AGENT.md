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
| `playwright-mcp` | MCP server | Automatisation navigateur via Playwright |
| `context7-mcp` | MCP server | Documentation à jour des librairies |
| `sequential-thinking-mcp` | MCP server | Raisonnement structuré étape par étape |

## Ajouter un plugin

1. Créer `plugins/<nom>/.claude-plugin/plugin.json`
2. Créer `plugins/<nom>/skills/<nom>/SKILL.md`
3. Ajouter l'entrée dans `.claude-plugin/marketplace.json` (voir schéma ci-dessous)
4. Mettre à jour `README.md` (tableau des plugins)
5. Mettre à jour ce fichier (tableau des plugins)

## Schéma marketplace.json

Fichier : `.claude-plugin/marketplace.json`

### Structure globale

```json
{
  "$schema": "https://code.claude.com/schemas/marketplace.json",
  "name": "liksi-tools",
  "owner": { "name": "Liksi", "email": "tech@liksi.fr" },
  "description": "...",
  "plugins": [ ... ]
}
```

### Entrée plugin (obligatoire : `name` + `source`)

```json
{
  "name": "mon-plugin",
  "source": "./plugins/mon-plugin",
  "description": "Ce que fait le plugin",
  "version": "1.0.0",
  "category": "content",
  "author": { "name": "Liksi" }
}
```

### Règles

- `name` : kebab-case minuscule, identique au nom du dossier
- `source` : chemin relatif depuis la racine du repo (commence par `./plugins/`)
- `category` : **toujours renseigner**, en minuscule. Valeurs utilisées : `content`, `marketing`
- `version` : bump obligatoire à chaque release (sinon les utilisateurs ne reçoivent pas la mise à jour)
- Ne pas mettre de champ `skills` : les skills sont auto-découverts depuis `./skills/` dans le plugin
- `description` et `version` dans `plugin.json` doivent être cohérents avec l'entrée marketplace

### Champs optionnels disponibles

| Champ | Type | Usage |
|-------|------|-------|
| `homepage` | string | URL de documentation |
| `repository` | string | URL du dépôt source |
| `license` | string | Identifiant SPDX (ex: `MIT`) |
| `keywords` | array | Tags pour la recherche |
| `tags` | array | Tags supplémentaires |

## Générer le frontend du marketplace

```bash
npx claude-marketplace-browser --marketplace-path . --marketplace-url git@github.com:liksi/claude-plugins.git
```

Produit un site statique dans `dist/` (ignoré par git).

## Déploiement local d'un skill

Copier le `SKILL.md` dans `~/.claude/skills/<nom>/SKILL.md` pour l'activer dans Claude Code.
