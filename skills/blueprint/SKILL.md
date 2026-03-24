---
name: blueprint
description: Use when creating, editing, or reviewing WordPress Playground blueprint JSON files. Triggers on mentions of blueprints, playground configuration, or requests to set up a WordPress demo environment.
---

# WordPress Playground Blueprints

## Overview

A Blueprint is a JSON file that declaratively configures a WordPress Playground instance — installing plugins/themes, setting options, running PHP/SQL, manipulating files, and more.

**Core principle:** Blueprints are trusted JSON-only declarations. No arbitrary JavaScript. They work on web, Node.js, and CLI.

## Quick Start Template

```json
{
  "$schema": "https://playground.wordpress.net/blueprint-schema.json",
  "landingPage": "/wp-admin/",
  "preferredVersions": { "php": "8.3", "wp": "latest" },
  "steps": [{ "step": "login" }]
}
```

**Always include:** `$schema` (enables validation/autocomplete) and `preferredVersions` (explicit is better than implicit).

## Top-Level Properties

All optional. `additionalProperties: false` — no invented keys.

| Property | Type | Notes |
|----------|------|-------|
| `$schema` | string | Always `"https://playground.wordpress.net/blueprint-schema.json"` |
| `landingPage` | string | Relative path, e.g. `/wp-admin/` |
| `meta` | object | `{ title, author, description?, categories? }` — title and author required |
| `preferredVersions` | object | `{ php, wp }` — both required when present |
| `features` | object | `{ networking?: boolean, intl?: boolean }` |
| `extraLibraries` | array | Only `"wp-cli"` currently. Auto-included if blueprint has `wp-cli` steps |
| `constants` | object | Shorthand for `defineWpConfigConsts`. Values: string/boolean/number |
| `plugins` | array | Shorthand for `installPlugin` steps. Strings = wp.org slugs |
| `siteOptions` | object | Shorthand for `setSiteOptions` |
| `login` | boolean or object | `true` = admin/password. Object = `{ username, password }` |
| `steps` | array | Main execution pipeline. Runs after shorthands |

### preferredVersions Values

- **php:** `"8.5"`, `"8.4"`, `"8.3"`, `"8.2"`, `"8.1"`, `"8.0"`, `"7.4"`, or `"latest"`. NO minor versions (e.g. `"7.4.1"` is invalid).
- **wp:** Last six major versions (e.g. `"6.3"`–`"6.8"`), `"latest"`, `"nightly"`, `"beta"`, or a URL to a custom zip.

### Shorthands vs Steps

Shorthands (`login`, `plugins`, `siteOptions`, `constants`) are expanded and prepended to `steps` in **arbitrary order**. Use explicit steps when execution order matters.

## Resource References

Resources tell Playground where to find files. Used by `installPlugin`, `installTheme`, `writeFile`, `writeFiles`, `importWxr`, etc.

| Resource Type | Required Fields | Example |
|--------------|----------------|---------|
| `wordpress.org/plugins` | `slug` | `{ "resource": "wordpress.org/plugins", "slug": "woocommerce" }` |
| `wordpress.org/themes` | `slug` | `{ "resource": "wordpress.org/themes", "slug": "astra" }` |
| `url` | `url` | `{ "resource": "url", "url": "https://example.com/plugin.zip" }` |
| `git:directory` | `url`, `ref` | See below |
| `literal` | `name`, `contents` | `{ "resource": "literal", "name": "file.txt", "contents": "hello" }` |
| `literal:directory` | `name`, `files` | See below |
| `zip` | `inner` | Wraps any reference in a ZIP |

### git:directory — Installing from GitHub

```json
{
  "resource": "git:directory",
  "url": "https://github.com/WordPress/gutenberg",
  "ref": "trunk",
  "refType": "branch",
  "path": "/"
}
```

- When using a branch or tag name for `ref`, you **must** set `refType` (`"branch"` | `"tag"` | `"commit"` | `"refname"`). Without it, only `"HEAD"` resolves reliably.
- `path` selects a subdirectory (defaults to repo root).

### literal:directory — Inline File Trees

```json
{
  "resource": "literal:directory",
  "name": "my-plugin",
  "files": {
    "plugin.php": "<?php /* Plugin Name: My Plugin */ ?>",
    "includes": {
      "helper.php": "<?php // helper code ?>"
    }
  }
}
```

- `files` uses nested objects for subdirectories — keys are filenames or directory names, values are strings (content) or objects (subdirectories).
- **Do NOT use path separators in keys** (e.g. `"includes/helper.php"` is wrong — use a nested `"includes": { "helper.php": "..." }` object).

## Steps Reference

Every step requires `"step": "<name>"`. Optional `progress: { weight, caption }` on any step.

### Plugin & Theme Installation

```json
{
  "step": "installPlugin",
  "pluginData": { "resource": "wordpress.org/plugins", "slug": "gutenberg" },
  "options": { "activate": true, "targetFolderName": "gutenberg" },
  "ifAlreadyInstalled": "overwrite"
}
```

```json
{
  "step": "installTheme",
  "themeData": { "resource": "wordpress.org/themes", "slug": "twentytwentyfour" },
  "options": { "activate": true, "importStarterContent": true },
  "ifAlreadyInstalled": "overwrite"
}
```

- Use `pluginData` / `themeData` — **NOT** the deprecated `pluginZipFile` / `themeZipFile`.
- `options.activate` controls activation. No need for a separate `activatePlugin`/`activateTheme` step when using `installPlugin`/`installTheme`.
- `ifAlreadyInstalled`: `"overwrite"` | `"skip"` | `"error"`

### Activation (standalone)

Only needed for plugins/themes already on disk (e.g. after `writeFile`/`writeFiles`):

```json
{ "step": "activatePlugin", "pluginPath": "my-plugin/my-plugin.php" }
{ "step": "activateTheme", "themeFolderName": "twentytwentyfour" }
```

### File Operations

```json
{ "step": "writeFile", "path": "/wordpress/wp-content/mu-plugins/custom.php", "data": "<?php // code" }
```

```json
{
  "step": "writeFiles",
  "writeToPath": "/wordpress/wp-content/plugins/",
  "filesTree": {
    "resource": "literal:directory",
    "name": "my-plugin",
    "files": {
      "plugin.php": "<?php\n/*\nPlugin Name: My Plugin\n*/",
      "includes": {
        "helpers.php": "<?php // helpers"
      }
    }
  }
}
```

**`writeFiles` requires a DirectoryReference** (`literal:directory` or `git:directory`) as `filesTree` — not a plain object.

Other file operations: `mkdir`, `cp`, `mv`, `rm`, `rmdir`, `unzip`.

### Running Code

**runPHP:**
```json
{ "step": "runPHP", "code": "<?php require '/wordpress/wp-load.php'; update_option('key', 'value');" }
```
**GOTCHA:** You must `require '/wordpress/wp-load.php';` to use any WordPress functions.

**wp-cli:**
```json
{ "step": "wp-cli", "command": "wp post create --post_type=page --post_title='Hello' --post_status=publish" }
```
The step name is `wp-cli` (with hyphen), NOT `cli` or `wpcli`.

**runSql:**
```json
{ "step": "runSql", "sql": { "resource": "literal", "name": "q.sql", "contents": "UPDATE wp_options SET option_value='val' WHERE option_name='key';" } }
```

### Site Configuration

```json
{ "step": "setSiteOptions", "options": { "blogname": "My Site", "blogdescription": "A tagline" } }
{ "step": "defineWpConfigConsts", "consts": { "WP_DEBUG": true } }
{ "step": "setSiteLanguage", "language": "en_US" }
{ "step": "defineSiteUrl", "siteUrl": "https://example.com" }
```

### Other Steps

| Step | Key Properties |
|------|---------------|
| `login` | `username?` (defaults to `"admin"`) |
| `enableMultisite` | (no required props) |
| `importWxr` | `file` (FileReference) |
| `importThemeStarterContent` | `themeSlug?` |
| `importWordPressFiles` | `wordPressFilesZip`, `pathInZip?` |
| `request` | `request: { url, method?, headers?, body? }` |
| `updateUserMeta` | `userId`, `meta` |
| `resetData` | (no props) |
| `runWpInstallationWizard` | `options: { adminUsername?, adminPassword? }` |

## Common Patterns

### Inline mu-plugin (auto-loads, no activation needed)
```json
{
  "step": "writeFile",
  "path": "/wordpress/wp-content/mu-plugins/custom.php",
  "data": "<?php\nadd_filter('show_admin_bar', '__return_false');"
}
```

### Inline plugin with multiple files
```json
[
  {
    "step": "writeFiles",
    "writeToPath": "/wordpress/wp-content/plugins/",
    "filesTree": {
      "resource": "literal:directory",
      "name": "my-plugin",
      "files": {
        "my-plugin.php": "<?php\n/*\nPlugin Name: My Plugin\n*/",
        "includes": { "helpers.php": "<?php // helpers" }
      }
    }
  },
  { "step": "activatePlugin", "pluginPath": "my-plugin/my-plugin.php" }
]
```

### Plugin from GitHub branch
```json
{
  "step": "installPlugin",
  "pluginData": {
    "resource": "git:directory",
    "url": "https://github.com/WordPress/gutenberg",
    "ref": "trunk",
    "refType": "branch",
    "path": "/"
  },
  "options": { "activate": true }
}
```

## Common Mistakes

| Mistake | Correct |
|---------|---------|
| Schema URL with `.org` | `playground.wordpress.net` |
| `pluginZipFile` / `themeZipFile` | `pluginData` / `themeData` |
| `"step": "cli"` | `"step": "wp-cli"` |
| Flat object as `writeFiles.filesTree` | Must be a `literal:directory` or `git:directory` resource |
| Path separators in `files` keys | Use nested objects for subdirectories |
| `runPHP` without `wp-load.php` | Always `require '/wordpress/wp-load.php';` for WP functions |
| Invented top-level keys | Schema is `additionalProperties: false` — only documented keys work |
| Inventing proxy URLs for GitHub | Use `git:directory` resource type |
| Omitting `refType` with branch/tag `ref` | Required — only `"HEAD"` works without it |

## Testing Blueprints

After creating or modifying a blueprint, test it. Two approaches are available depending on available tools.

### Option A: Browser Testing with Playground MCP

Use the Playground website at `https://playground.wordpress.net` combined with the **Playground MCP** and a browser MCP (Playwright or DevTools) to verify blueprints visually.

1. **Load the blueprint** — Pass the blueprint as a URL-encoded JSON in the hash fragment:
   ```
   https://playground.wordpress.net/#{"steps":[{"step":"login"}]}
   ```
2. **Use the Playground MCP** to interact with the running WordPress instance — execute PHP, read files, make HTTP requests, and navigate pages.
3. **Use a browser MCP** (Playwright or DevTools) to visually inspect the result — check sidebar menus, plugin lists, settings pages, or page content rendered by the blueprint.

### Option B: Local CLI Testing

Use the `wordpress-playground-server` skill to start a local Playground instance with the blueprint:

1. Start a server with `--blueprint /path/to/blueprint.json`
2. Use Playwright MCP to navigate and verify the expected state
3. Stop the server when done

For headless/CI validation without a UI:
```bash
npx @wp-playground/cli run-blueprint --blueprint=/path/to/blueprint.json
```

