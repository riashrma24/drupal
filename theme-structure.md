# Drupal Custom Theme Reference

> A comprehensive guide to Drupal custom theming — file structure, Twig templating, libraries, preprocess hooks, and more.

---

## Table of Contents

1. [Theme File Structure](#1-theme-file-structure)
2. [Core Template Files](#2-core-template-files)
3. [info.yml Configuration](#3-infoyml-configuration)
4. [Libraries API](#4-libraries-api)
5. [CSS SMACSS Weight Order](#5-css-smacss-weight-order)
6. [Attaching Libraries](#6-attaching-libraries)
7. [Regions and Blocks](#7-regions-and-blocks)
8. [Twig Templating Engine](#8-twig-templating-engine)
9. [Twig Syntax Reference](#9-twig-syntax-reference)
10. [Filters and Functions](#10-filters-and-functions)
11. [Template Inheritance](#11-template-inheritance)
12. [Drupal Twig Extensions](#12-drupal-twig-extensions)
13. [Template Suggestions](#13-template-suggestions)
14. [Preprocess Hooks](#14-preprocess-hooks)
15. [Breakpoints](#15-breakpoints)
16. [FAQ](#16-faq)

---

## 1. Theme File Structure

```
/web/themes/custom/mytheme/
│
├── mytheme.info.yml
├── mytheme.libraries.yml
├── mytheme.theme
├── mytheme.breakpoints.yml
├── screenshot.png
│
├── css/
│   ├── base/
│   │   ├── reset.css
│   │   ├── typography.css
│   │   └── variables.css
│   ├── components/
│   │   ├── button.css
│   │   ├── card.css
│   │   └── navigation.css
│   └── layouts/
│       ├── header.css
│       ├── footer.css
│       └── grid.css
│
├── js/
│   └── mytheme.js
│
├── images/
│   ├── logo.svg
│   └── banner.jpg
│
├── fonts/
│
└── templates/
    ├── layout/
    │   ├── html.html.twig
    │   ├── page.html.twig
    │   └── region.html.twig
    ├── navigation/
    │   ├── menu.html.twig
    │   └── breadcrumbs.html.twig
    ├── content/
    │   ├── node.html.twig
    │   ├── field.html.twig
    │   ├── views-view.html.twig
    │   └── paragraph.html.twig
    └── block/
        └── block.html.twig
```

---

## 2. Core Template Files

| # | Template File | Purpose | Variants |
|---|---------------|---------|---------|
| 1 | `html.html.twig` | Full HTML document wrapper (`doctype`, `<head>`, `<body>`) | — |
| 2 | `page.html.twig` | Page layout — regions, header, footer | `page--front.html.twig`, etc. |
| 3 | `node.html.twig` | Content node wrapper | `node--article.html.twig` |
| 4 | `block.html.twig` | Block wrapper | `block--system-menu-block.html.twig` |
| 5 | `field.html.twig` | Field output wrapper | — |
| 6 | `views-view.html.twig` | Views container | — |

---

## 3. info.yml Configuration

Every Drupal theme requires a `mytheme.info.yml` file declaring metadata, regions, libraries, and more.

```yaml
name: My Theme
type: theme
description: 'Custom theme for My Site'
package: Custom

core_version_requirement: ^10 || ^11

# Base theme options: 'stable9', 'classy', or a contributed base theme
base theme: stable9

# Libraries attached to every page
libraries:
  - mytheme/global-styling
  - mytheme/global-scripts

# Override libraries from other themes/modules
libraries-override:
  classy/base: false   # Disable Classy's base CSS

# Extend libraries from other themes/modules
libraries-extend:
  core/drupal.dialog:
    - mytheme/dialog-overrides

# Regions — appear in Block Layout UI
regions:
  header: Header
  primary_menu: Primary Menu
  highlighted: Highlighted
  content: Content
  sidebar_first: Sidebar First
  sidebar_second: Sidebar Second
  footer: Footer
```

---

## 4. Libraries API

### `mytheme.libraries.yml`

```yaml
global-styling:
  version: VERSION
  css:
    base:
      css/base/reset.css: {}
      css/base/variables.css: {}
    component:
      css/components/card.css: {}
    theme:
      css/theme/colors.css: {}

global-scripts:
  js:
    js/mytheme.js: {}
  dependencies:
    - core/jquery
    - core/drupal
    - core/once

# Component-specific library — attach only where needed
slider:
  css:
    component:
      css/components/slider.css: {}
  js:
    js/slider.js:
      defer: true
  dependencies:
    - core/drupal
```

### Library Structure Overview

```
mytheme.libraries.yml
│
├── global-styling
│   └── CSS
│       ├── base/       → reset.css, variables.css
│       ├── component/  → card.css
│       └── theme/      → colors.css
│
├── global-scripts
│   ├── JS              → mytheme.js
│   └── Dependencies    → core/jquery, core/drupal, core/once
│
└── slider
    ├── CSS             → slider.css
    ├── JS              → slider.js (defer: true)
    └── Dependencies    → core/drupal
```

---

## 5. CSS SMACSS Weight Order

| Key | Weight | Use For |
|-----|--------|---------|
| `base` | -200 | HTML element resets, normalize, variables |
| `layout` | -100 | Grid systems, page-level layout |
| `component` | 0 | Reusable UI components (cards, menus) |
| `state` | 100 | States — `.is-active`, `.is-collapsed` |
| `theme` | 200 | Visual overrides, color themes |

---

## 6. Attaching Libraries

### In a `.html.twig` file

```twig
{# Attach a library to this template #}
{{ attach_library('mytheme/slider') }}

{# Conditionally attach #}
{% if content.field_gallery %}
  {{ attach_library('mytheme/gallery') }}
{% endif %}
```

### In a preprocess hook

```php
function mytheme_preprocess_node(&$variables)
{
    if ($variables['node']->bundle() === 'article')
    {
        $variables['#attached']['library'][] = 'mytheme/article-styles';
    }
}
```

---

## 7. Regions and Blocks

Regions are named areas defined in `.info.yml`. Blocks are placed into regions via the Block Layout UI or config. In Twig, regions render as variables.

### `page.html.twig` — example layout

```twig
<header class="site-header">
  {% if page.header %}
    <div class="region region--header">
      {{ page.header }}
    </div>
  {% endif %}
</header>

<main class="layout-main">
  <div class="layout-container">
    {% if page.sidebar_first %}
      <aside class="layout-sidebar-first">
        {{ page.sidebar_first }}
      </aside>
    {% endif %}

    <div class="layout-content">
      {{ page.content }}
    </div>
  </div>
</main>
```

> **Tip:** Always wrap region renders in `{% if page.region_name %}` checks. This lets editors remove all blocks from a region without breaking the layout.

---

## 8. Twig Templating Engine

Twig is a fast, secure, flexible PHP templating engine. In Drupal, every `.html.twig` file compiles to plain PHP and is cached. It enforces separation of concerns — no raw PHP in templates.

### Delimiter Types

| Delimiter | Type | Description |
|-----------|------|-------------|
| `{{ ... }}` | Print | Outputs a variable or expression. Auto-escapes HTML. |
| `{% ... %}` | Tag | Control flow — `if`, `for`, `set`, `block`, `extends`, `include`, etc. |
| `{# ... #}` | Comment | Not rendered. Not sent to the browser. |

```twig
{# This is a comment — not rendered #}

{# Print a variable #}
<h1>{{ title }}</h1>

{# Control flow #}
{% if user.isAuthenticated() %}
  <p>Welcome, {{ user.name }}</p>
{% else %}
  <p>Please <a href="/login">log in</a></p>
{% endif %}
```

> **Security:** Never use `|raw` on user input. Only apply it to content that Drupal's render system has already sanitized — like `{{ content.body|raw }}` when `body` is a render array Drupal has processed. Even then, prefer rendering through Drupal's render system.

---

## 9. Twig Syntax Reference

| Operator | Description | Example |
|----------|-------------|---------|
| `== != < > <= >=` | Comparison | `{% if score >= 10 %}` |
| `and`, `or`, `not` | Logical | `{% if a and not b %}` |
| `in` | Containment | `{% if 'php' in tags %}` |
| `is defined` | Variable exists | `{% if var is defined %}` |
| `is empty` | Empty check | `{% if list is empty %}` |
| `is odd` / `is even` | Number parity | `{% if loop.index is odd %}` |
| `~` | String concat | `{{ 'Hello ' ~ name }}` |
| `..` | Range | `{% for i in 1..5 %}` |
| `\|` | Apply filter | `{{ name\|upper }}` |

---

## 10. Filters and Functions

Filters transform variables using the pipe `|` syntax. Functions generate values. Both include core Twig and Drupal-specific additions.

### Core Twig Filters

| Filter | Example | Output |
|--------|---------|--------|
| `upper` / `lower` | `{{ 'hello'\|upper }}` | `HELLO` |
| `capitalize` | `{{ 'hello world'\|capitalize }}` | `Hello world` |
| `title` | `{{ 'hello world'\|title }}` | `Hello World` |
| `trim` | `{{ ' hi '\|trim }}` | `hi` |
| `length` | `{{ items\|length }}` | integer count |
| `join` | `{{ items\|join(', ') }}` | `a, b, c` |
| `split` | `{{ 'a,b'\|split(',') }}` | `['a','b']` |
| `replace` | `{{ 'a-b'\|replace({'-': '_'}) }}` | `a_b` |
| `slice` | `{{ items\|slice(0, 3) }}` | first 3 items |
| `sort` | `{{ items\|sort }}` | sorted array |
| `keys` | `{{ obj\|keys }}` | array of keys |
| `first` / `last` | `{{ items\|first }}` | first element |
| `reverse` | `{{ items\|reverse }}` | reversed array |
| `number_format` | `{{ 1234.5\|number_format(2) }}` | `1,234.50` |
| `date` | `{{ date\|date('Y-m-d') }}` | `2024-06-01` |
| `raw` | `{{ markup\|raw }}` | unescaped HTML |
| `default` | `{{ val\|default('N/A') }}` | `N/A` if falsy |
| `escape` (`e`) | `{{ val\|e('html') }}` | HTML-escaped |
| `striptags` | `{{ html\|striptags }}` | text only |
| `nl2br` | `{{ text\|nl2br }}` | `\n` → `<br>` |
| `batch` | `{{ items\|batch(3) }}` | grouped chunks |

### Drupal-Specific Filters

| Filter | Purpose |
|--------|---------|
| `t` | Translate string: `{{ 'Hello'\|t }}` |
| `clean_class` | Make string safe as a CSS class |
| `clean_id` | Make string safe as an HTML ID |
| `format_date` | Format a Drupal timestamp |
| `without` | Render array minus specified keys: `{{ content\|without('field_image') }}` |
| `render` | Force render of a render array |
| `safe_join` | Join with an XSS-safe separator |
| `placeholder` | Wrap in `<em>` for use in `t()` strings |

### Examples

```twig
{# Translation #}
{{ 'Read more'|t }}
{{ 'Hello @name'|t({'@name': user.name}) }}

{# CSS classes — clean_class prevents invalid chars #}
{% set classes = [
  'node',
  'node--type-' ~ node.bundle()|clean_class,
  node.isPromoted() ? 'node--promoted',
] %}
<article class="{{ classes|join(' ') }}">

{# Render content but exclude certain fields #}
{{ content|without('field_image', 'field_tags') }}

{# Render a specific field only #}
{{ content.field_image }}
```

---

## 11. Template Inheritance

Twig's most powerful feature. A child template `extends` a parent, overriding named `{% block %}` sections. This creates DRY, composable layouts.

```
base.html.twig  ←  layout.html.twig  ←  node--article.html.twig
```

### Parent template — `base.html.twig`

```twig
<!DOCTYPE html>
<html lang="{{ language }}">
<head>
  {% block head %}
    <meta charset="utf-8">
    {{ head }}
  {% endblock %}
</head>
<body class="{% block body_classes %}{% endblock %}">
  {% block content %}{% endblock %}
  {% block footer %}
    <footer>Default footer</footer>
  {% endblock %}
</body>
</html>
```

### Child template — extends and overrides blocks

```twig
{% extends 'mytheme/templates/base.html.twig' %}

{% block body_classes %}page--article{% endblock %}

{% block content %}
  <main>
    <h1>{{ title }}</h1>
    {{ content }}
  </main>
{% endblock %}

{% block footer %}
  {# Call parent's footer, then add more #}
  {{ parent() }}
  <div class="extra-footer">Extra content</div>
{% endblock %}
```

### `include` vs `embed` vs `extends`

| Tag | Use When | Overridable Blocks? |
|-----|----------|---------------------|
| `include` | Reuse a partial (card, icon, etc.) with variable passing | No |
| `extends` | Child inherits full structure, overrides named blocks | Yes |
| `embed` | Include + override blocks inside the included template | Yes |
| `use` | Import blocks from another template without extending | Partial |

### `include` with variable passing

```twig
{% include 'mytheme/templates/partials/card.html.twig' with {
  'title': node.label(),
  'image': content.field_image,
  'classes': ['card--featured']
} only %}
{# 'only' keyword: the partial only gets the vars you pass #}
```

---

## 12. Drupal Twig Extensions

Drupal adds its own functions, filters, and global variables to Twig beyond vanilla Twig.

```twig
{# Attach a library #}
{{ attach_library('mytheme/slider') }}

{# Generate a URL (absolute) #}
<a href="{{ url('entity.node.canonical', {'node': node.id}) }}">Read more</a>

{# Generate a path (relative) #}
<a href="{{ path('<front>') }}">Home</a>

{# Link render element #}
{{ link('Home', '<front>', {'class': ['home-link']}) }}

{# create_attribute — build dynamic attribute objects #}
{% set attrs = create_attribute({'class': ['card'], 'id': 'card-1'}) %}
<div {{ attrs }}>...</div>

{# Using the built-in attributes variable #}
<article {{ attributes.addClass('custom-class') }}>
<article {{ attributes.setAttribute('data-id', node.id()) }}>
```

### Global Variables

Available in **all** templates:

| Variable | Description |
|----------|-------------|
| `{{ is_front }}` | `true` on the front page |
| `{{ is_admin }}` | `true` on admin pages |
| `{{ logged_in }}` | `true` if user is logged in |
| `{{ language }}` | Current language object |
| `{{ user }}` | Current user account object |
| `{{ directory }}` | Theme's directory path (for assets) |
| `{{ base_path }}` | Drupal base path (usually `/`) |
| `{{ db_offline }}` | `true` if site is in maintenance mode |

### Practical Patterns with Globals

```twig
{# Conditional for front page #}
{% if is_front %}
  <h1>{{ site_name }}</h1>
{% else %}
  <a href="{{ path('<front>') }}"><span>{{ site_name }}</span></a>
{% endif %}

{# Asset path using directory variable #}
<img src="{{ base_path ~ directory }}/images/logo.svg" alt="Logo">

{# Admin-only debug block #}
{% if is_admin %}
  <pre>{{ node|json_encode(constant('JSON_PRETTY_PRINT')) }}</pre>
{% endif %}
```

---

## 13. Template Suggestions

Drupal builds a list of template candidates in specificity order — most specific first. Your theme can override any template without touching core.

> **Tip:** Enable Twig debug in `services.yml` to see all template suggestions in your browser's HTML source comments.
> Set `twig.config: debug: true`

### Node Templates — Specificity Order

| Template File | Matches |
|---------------|---------|
| `node--123--full.html.twig` | Node ID 123, full view mode |
| `node--123.html.twig` | Node ID 123, any view mode |
| `node--article--full.html.twig` | Article type, full view mode |
| `node--article.html.twig` | Any article node |
| `node--full.html.twig` | Any node, full view mode |
| `node.html.twig` | Any node (fallback) |

### Other Common Suggestion Patterns

| Template | Specificity Pattern |
|----------|---------------------|
| Block | `block--[module]--[delta].html.twig` |
| Page | `page--[path].html.twig`, `page--front.html.twig` |
| Field | `field--[field_name]--[bundle].html.twig` |
| View | `views-view--[view_name]--[display_id].html.twig` |
| Menu | `menu--[menu_name].html.twig` |

### Custom Template Suggestions via Hook

```php
function mytheme_theme_suggestions_node_alter(&$suggestions, $variables)
{
    $node = $variables['elements']['#node'];

    // Add suggestion based on paragraph field reference
    if ($node->hasField('field_layout')) {
        $layout = $node->get('field_layout')->value;
        $suggestions[] = 'node__' . $node->bundle() . '__' . $layout;
    }
}
```

---

## 14. Preprocess Hooks

Preprocess functions run before a template renders. They live in `mytheme.theme` and add or modify variables that Twig can access.

### Hook Naming Convention

| Hook | Fires Before |
|------|-------------|
| `hook_preprocess_html()` | `html.html.twig` |
| `hook_preprocess_page()` | `page.html.twig` |
| `hook_preprocess_node()` | `node.html.twig` (all nodes) |
| `hook_preprocess_block()` | `block.html.twig` |
| `hook_preprocess_field()` | `field.html.twig` |
| `hook_preprocess_views_view()` | `views-view.html.twig` |
| `hook_preprocess_paragraph()` | `paragraph.html.twig` |

### Examples

```php
use Drupal\Core\Url;

/**
 * Implements hook_preprocess_node().
 */
function mytheme_preprocess_node(&$variables)
{
    $node = $variables['node'];

    // 1. Add a computed variable
    $variables['reading_time'] = ceil(
        str_word_count(strip_tags($node->get('body')->value)) / 200
    );

    // 2. Build a classes array
    $variables['classes'] = [
        'node',
        'node--type-' . $node->bundle(),
        $node->isPromoted() ? 'node--promoted' : '',
    ];

    // 3. Add a URL object
    $variables['node_url'] = $node->toUrl()->toString();

    // 4. Load a related entity
    if (!$node->get('field_author')->isEmpty()) {
        $author = $node->get('field_author')->entity;
        $variables['author_name'] = $author->label();
    }
}

/**
 * Implements hook_preprocess_html().
 */
function mytheme_preprocess_html(&$variables)
{
    // Add node type as body class
    $node = \Drupal::routeMatch()->getParameter('node');
    if ($node) {
        $variables['attributes']['class'][] = 'page-node-type--' . $node->bundle();
    }
}
```

---

## 15. Breakpoints

Drupal's Breakpoint module allows themes to declare breakpoints in YAML. They're used by responsive image styles and Picture elements.

### `mytheme.breakpoints.yml`

```yaml
mytheme.mobile:
  label: Mobile
  mediaQuery: ''
  weight: 0
  multipliers:
    - 1x
    - 2x

mytheme.tablet:
  label: Tablet
  mediaQuery: '(min-width: 768px)'
  weight: 1
  multipliers:
    - 1x
    - 2x

mytheme.desktop:
  label: Desktop
  mediaQuery: '(min-width: 1200px)'
  weight: 2
  multipliers:
    - 1x
    - 2x
```

After defining breakpoints, create **Responsive Image Styles** at:
`Admin → Configuration → Media → Responsive image styles`

Map each breakpoint to an image style, then use the responsive image formatter on image fields.

---

## 16. FAQ

**Q: What is the difference between `.html.twig` and `.twig` files?**

In Drupal theming, `.html.twig` is the conventional extension for templates that produce HTML markup — this is the standard for all Drupal core and contributed module templates (e.g. `node.html.twig`, `page.html.twig`). The double extension makes it immediately clear that the file is both an HTML document and a Twig template.

A plain `.twig` extension can also be used, and Twig itself treats both identically at the engine level. However, Drupal's template suggestion system and module conventions rely on the `.html.twig` pattern, so you should always use `.html.twig` for your theme templates to ensure correct discovery, overrides, and debugging output.

---

*Last updated: Drupal 10/11 compatible*
