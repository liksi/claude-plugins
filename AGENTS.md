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

## Validation obligatoire

Après **toute modification** (marketplace.json, plugin.json, SKILL.md, nouvelle entrée…), valider avec :

```bash
/plugin validate .
```

La commande doit se terminer sans erreur avant tout commit.

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

## Schéma plugin.json

Fichier : `plugins/<nom>/.claude-plugin/plugin.json`

Le manifest est optionnel — Claude Code auto-découvre les composants dans les emplacements par défaut. L'utiliser pour fournir des métadonnées ou des chemins personnalisés.

### Champ obligatoire

| Champ  | Type   | Description                                    |
|--------|--------|------------------------------------------------|
| `name` | string | Identifiant unique en kebab-case, sans espaces |

### Métadonnées

| Champ         | Type   | Description                                                        |
|---------------|--------|--------------------------------------------------------------------|
| `$schema`     | string | URL du JSON Schema pour l'autocomplétion dans l'éditeur            |
| `displayName` | string | Nom lisible affiché dans l'UI (espaces autorisés, ≥ v2.1.143)     |
| `version`     | string | Version sémantique — à bumper pour déclencher une MAJ utilisateur |
| `description` | string | Description courte du plugin                                       |
| `author`      | object | `{ name, email, url }`                                             |
| `homepage`    | string | URL de documentation                                               |
| `repository`  | string | URL du dépôt source                                                |
| `license`     | string | Identifiant SPDX (`MIT`, `Apache-2.0`…)                           |
| `keywords`    | array  | Tags pour la recherche                                             |

### Chemins des composants

| Champ                   | Type                  | Comportement                                                                              |
|-------------------------|-----------------------|-------------------------------------------------------------------------------------------|
| `skills`                | string\|array         | **Ajoute** aux dossiers par défaut (`skills/` est toujours scanné en plus)               |
| `commands`              | string\|array         | **Remplace** `commands/` par défaut                                                       |
| `agents`                | string\|array         | **Remplace** `agents/` par défaut                                                         |
| `hooks`                 | string\|array\|object | Chemin(s) vers config hooks ou config inline                                              |
| `mcpServers`            | string\|array\|object | Chemin(s) vers config MCP ou config inline (`command`+`args` uniquement dans marketplace) |
| `lspServers`            | string\|array\|object | Config Language Server Protocol                                                           |
| `outputStyles`          | string\|array         | **Remplace** `output-styles/` par défaut                                                  |
| `experimental.themes`   | string\|array         | Thèmes de couleur (remplace `themes/`)                                                    |
| `experimental.monitors` | string\|array         | Moniteurs background démarrés automatiquement                                             |
| `userConfig`            | object                | Valeurs configurables par l'utilisateur à l'activation du plugin                         |
| `channels`              | array                 | Canaux de messagerie (Telegram, Slack…) injectés dans la conversation                    |
| `dependencies`          | array                 | Autres plugins requis, avec contraintes de version semver                                 |

Tous les chemins doivent être relatifs à la racine du plugin et commencer par `./`.

### Variables d'environnement disponibles dans les configs

| Variable                    | Description                                                  |
|-----------------------------|--------------------------------------------------------------|
| `${CLAUDE_PLUGIN_ROOT}`     | Chemin absolu du répertoire d'installation du plugin         |
| `${CLAUDE_PLUGIN_DATA}`     | Répertoire persistant pour l'état du plugin (survit aux MAJ) |
| `${CLAUDE_PROJECT_DIR}`     | Racine du projet courant                                     |
| `${user_config.<KEY>}`      | Valeur de configuration utilisateur déclarée dans `userConfig` |

### Exemple complet

```json
{
  "name": "mon-plugin",
  "displayName": "Mon Plugin",
  "version": "1.0.0",
  "description": "Ce que fait le plugin",
  "author": { "name": "Liksi" },
  "mcpServers": {
    "mon-serveur": {
      "command": "npx",
      "args": ["-y", "@org/mcp-server@latest"]
    }
  }
}
```

## Déploiement local d'un skill

Copier le `SKILL.md` dans `~/.claude/skills/<nom>/SKILL.md` pour l'activer dans Claude Code.
