A custom Drupal theme lives in /web/themes/custom/

mytheme/
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
│   │
│   ├── components/
│   │   ├── button.css
│   │   ├── card.css
│   │   └── navigation.css
│   │
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
    │
    ├── navigation/
    │   ├── menu.html.twig
    │   └── breadcrumbs.html.twig
    │
    ├── content/
    │   ├── node.html.twig
    │   ├── field.html.twig
    │   ├── views-view.html.twig
    │   └── paragraph.html.twig
    │
    └── block/
        └── block.html.twig


# Core template files : 
1. html.html.twig
   - Full HTML document wrapper (doctype, head, body)
   - html.html.twig
2. page.html.twig
   - Page layout — regions, header, footer
   - page--front.html.twig etc.
3. node.html.twig
   - Content node wrapper
   - node--article.html.twig
4. block.html.twig
   - Block wrapper
   - block--system-menu-block.html.twig
5. field.html.twig
   - Field output wrapper
6. views-view.html.twig
   - Views container
  

# info.yml file
Every Drupal theme needs a mytheme.info.yml file. It declares metadata, regions, libraries, and more.

name: My Theme
type: theme
description: 'Custom theme for My Site'
package: Custom

core_version_requirement: ^10 || ^11

// Base theme — 'stable9', 'classy', or a contributed base theme
base theme: stable9

// Libraries attached to every page
libraries:
  - mytheme/global-styling
  - mytheme/global-scripts

// Override libraries from other themes/modules
libraries-override:
  classy/base: false   // Disable Classy's base CSS

// Extend libraries from other themes/modules
libraries-extend:
  core/drupal.dialog:
    - mytheme/dialog-overrides

// Regions — appear in Block Layout UI
regions:
  header: Header
  primary_menu: Primary Menu
  highlighted: Highlighted
  content: Content
  sidebar_first: Sidebar First
  sidebar_second: Sidebar Second
  footer: Footer


Libraries API — CSS & JS

# mytheme.libraries.yml

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

# Component-specific library (attach only where needed)
slider:
  css:
    component:
      css/components/slider.css: {}

  js:
    js/slider.js:
      defer: true

  dependencies:
    - core/drupal

----

  mytheme.libraries.yml
│
├── global-styling
│   ├── CSS
│   │   ├── base
│   │   │   ├── reset.css
│   │   │   └── variables.css
│   │   ├── component
│   │   │   └── card.css
│   │   └── theme
│   │       └── colors.css
│
├── global-scripts
│   ├── JS
│   │   └── mytheme.js
│   └── Dependencies
│       ├── core/jquery
│       ├── core/drupal
│       └── core/once
│
└── slider
    ├── CSS
    │   └── slider.css
    ├── JS
    │   └── slider.js (defer=true)
    └── Dependencies
        └── core/drupal

