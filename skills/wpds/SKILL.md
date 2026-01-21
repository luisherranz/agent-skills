---
name: wpds
description: "Use when building UIs leveraging the WordPress Design System (WPDS) and its components, tokens, patterns, etc."
# compatibility: "Requires WPDS MCP server configured and running."
---

# WordPress Design System (WPDS)

## Prerequisites

This skill works best with the **WPDS MCP server** installed. The MCP provides access to WordPress Design System documentation and resources, such as components and DS token lists.

The following terms should be treated as synonyms:
- "WordPress" and "WP";
- "Design System" and "DS";
- "WordPress Design System" and "WPDS".

## When to use

Use this skill when the user mentions:

- building any UI in a WordPress-related context (for example, Gutenberg, WooCommerce, WordPress.com, Jetpack, etc etc);
- WordPress Design System, WPDS, Design System;
- UI components, Design tokens, color primitives, spacing scales, typography variables and presets;
- Specific component packages such as @wordpress/components or @wordpress/ui;

## Rules

### Use the WPDS MCP server to access WPDS-related documentation

- Use the WPDS MCP server to retrieve the canonical, authoritative documentation:
  - reference site (`wpds://reference` and `wpds://reference/:query`)
  - list of available components (`wpds://components`) and specific component information (`wpds://components/:name`)
  - list of available tokens (`wpds://tokens`)

### Required documentation

Before working on any WPDS-related tasks, make sure you read the  documentation on the reference site. This documentation should take the absolute precedence when evaluating the best course of action for any given tasks.

### Boundaries

- Don't spend too much time on non-UI related aspects of an answer (for example, fetching data from stores).
- Focus on building UI that adheres as much as possible to the WPDS best practices, uses the most fitting WPDS components/tokens/patterns.

### Tech stack

- Unless you are told otherwise (or gathered specific information from the local context of the request), assume the following tech stack: TypeScript, React, CSS.

### Boundaries

- Don't spend too much time on non-UI related aspects of an answer (for example, fetching data from stores).
- Focus on building UI that adheres as much as possible to the WPDS best practices, uses the most fitting WPDS components/tokens/patterns.

### Validation

- If the local context in which a task is running provide lint scripts, use them to validate the proposed code output when possible.