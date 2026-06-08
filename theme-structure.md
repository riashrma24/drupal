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

CSS SMACSS weight order

Key	        Weight	Use for
base	    -200	HTML element resets, normalize, variables
layout	    -100	Grid systems, page-level layout
component	0	    Reusable UI components (cards, menus)
state	    100	    States — .is-active, .is-collapsed
theme	    200	    Visual overrides, color themes

# Attaching libraries in twig

in a .html.twig file

{# Attach a library to this template #} 
{{ attach_library('mytheme/slider') }} 
{# Conditionally attach #} 
{% if content.field_gallery %} 
{{ attach_library('mytheme/gallery') }}
{% endif %}

in a preprocess hook

function mytheme_preprocess_node(&$variables) 
{
    if ($variables['node']->bundle() === 'article') 
    { 
        $variables['#attached']['library'][] = 'mytheme/article-styles'; 
    }
}

## Regions and Blocks

Regions are named areas defined in .info.yml. Blocks are placed into regions via the Block Layout UI or config. In Twig, regions render as variables

in page.html.twig

<header class="site-header">
    {% if page.header %}
        <div class="region region--header"> {{ page.header }} </div> 
    {% endif %} 
</header> 
<main class="layout-main">
    <div class="layout-container"> 
        {% if page.sidebar_first %}
        <aside class="layout-sidebar-first"> {{ page.sidebar_first }} </aside> 
        {% endif %} 
        <div class="layout-content"> 
            {{ page.content }}
        </div> 
</div> 
</main>

note : Always wrap region renders in {% if page.region_name %} checks. This lets editors remove all blocks from a region without breaking layout.

## Twig - Drupal's templating engine

Twig is a fast, secure, flexible PHP templating engine by Fabien Potencier. In Drupal, every .html.twig file compiles to plain PHP and is cached. It enforces separation of concerns — no raw PHP in templates.

Three delimiter types : 

Delimiter	Type	Description
{{ ... }}	Print	Outputs a variable or expression. Auto-escapes HTML.
{% ... %}	Tag	Control flow — if, for, set, block, extends, include, etc.
{# ... #}	Comment	Not rendered. Not sent to the browser. Use freely.

{# This is a comment — not rendered #} {# Print a variable #} <h1>{{ title }}</h1> {# Control flow tag #} {% if user.isAuthenticated() %} <p>Welcome, {{ user.name }}</p> {% else %} <p>Please <a href="/login">log in</a></p> {% endif %}

note : Never use |raw on user input. Only apply it to content that Drupal's render system has already sanitized — like {{ content.body|raw }} when body is a render array that Drupal has processed. Even then, prefer rendering through Drupal's render system.

## Twig syntax reference

Operator	        Description	    Example
== != < > <= >=	    Comparison	    {% if score >= 10 %}
and, or, not	    Logical	        {% if a and not b %}
in	                Containment	    {% if 'php' in tags %}
is defined	        Variable exists	{% if var is defined %}
is empty	        Empty check	    {% if list is empty %}
is odd / is even	Number parity	{% if loop.index is odd %}
~	                String concat	{{ 'Hello ' ~ name }}
..	                Range	        {% for i in 1..5 %}
|	                Apply filter	{{ name|upper }}


## Filters and functions

Filters transform variables using the pipe | syntax. Functions generate values. Both are core Twig plus Drupal-specific additions.

Core Twig filters
Filter	Example	Output
upper / lower	{{ 'hello'|upper }}	HELLO
capitalize	{{ 'hello world'|capitalize }}	Hello world
title	{{ 'hello world'|title }}	Hello World
trim	{{ ' hi '|trim }}	hi
length	{{ items|length }}	integer count
join	{{ items|join(', ') }}	a, b, c
split	{{ 'a,b'|split(',') }}	['a','b']
replace	{{ 'a-b'|replace({'-': '_'}) }}	a_b
slice	{{ items|slice(0, 3) }}	first 3
sort	{{ items|sort }}	sorted array
keys	{{ obj|keys }}	array of keys
first / last	{{ items|first }}	first element
reverse	{{ items|reverse }}	reversed array
number_format	{{ 1234.5|number_format(2) }}	1,234.50
date	{{ date|date('Y-m-d') }}	2024-06-01
raw	{{ markup|raw }}	unescaped HTML
default	{{ val|default('N/A') }}	N/A if falsy
escape (e)	{{ val|e('html') }}	HTML-escaped
striptags	{{ html|striptags }}	text only
nl2br	{{ text|nl2br }}	\n → <br>
batch	{{ items|batch(3) }}	grouped chunks


## Drupal-specific filters

Filter	Purpose
t	Translate string: {{ 'Hello'|t }}
clean_class	Make string safe as CSS class
clean_id	Make string safe as HTML id
format_date	Format Drupal timestamp
without	Render array minus specified keys: {{ content|without('field_image') }}
render	Force render of a render array
safe_join	Join with XSS-safe separator
placeholder	Wrap in <em> for use in t() strings

eg : 
{# Translation #} {{ 'Read more'|t }} {{ 'Hello @name'|t({'@name': user.name}) }} {# CSS classes — clean_class prevents invalid chars #} {% set classes = [ 'node', 'node--type-' ~ node.bundle()|clean_class, node.isPromoted() ? 'node--promoted', ] %} <article class="{{ classes|join(' ') }}"> {# Render content but exclude certain fields #} {{ content|without('field_image', 'field_tags') }} {# Render a specific field only #} {{ content.field_image }}

Template inheritance : 
Twig's most powerful feature. A child template extends a parent, overriding named blocks. This creates DRY, composable layouts.

base.html.twig ← extends layout.html.twig ←extends node--article.html.twig

base.html.twig — parent template
<!DOCTYPE html> <html lang="{{ language }}"> <head> {% block head %} <meta charset="utf-8"> {{ head }} {% endblock %} </head> <body class="{% block body_classes %}{% endblock %}"> {% block content %}{% endblock %} {% block footer %} <footer>Default footer</footer> {% endblock %} </body> </html>

child.html.twig — extends and overrides blocks
{% extends 'mytheme/templates/base.html.twig' %} {% block body_classes %}page--article{% endblock %} {% block content %} <main> <h1>{{ title }}</h1> {{ content }} </main> {% endblock %} {% block footer %} {# Call parent's footer, then add more #} {{ parent() }} <div class="extra-footer">Extra content</div> {% endblock %}

include vs embed vs extends

Tag	Use when	Overridable blocks?
include	Reuse a partial (card, icon, etc.) with variable passing	No
extends	Child inherits full structure, overrides named blocks	Yes
embed	Include + override blocks inside the included template	Yes
use	Import blocks from another template without extending	Partial

include — passing variables
{% include 'mytheme/templates/partials/card.html.twig' with { 'title': node.label(), 'image': content.field_image, 'classes': ['card--featured'] } only %} {# 'only' keyword: the partial only gets the vars you pass #}

Drupal Twig Extensions : 
Drupal adds its own functions, filters, and global variables to Twig that are not in vanilla Twig.

{# Attach a library #} {{ attach_library('mytheme/slider') }} {# Generate a URL #} <a href="{{ url('entity.node.canonical', {'node': node.id}) }}">Read more</a> {# Generate a path (relative) #} <a href="{{ path('<front>') }}">Home</a> {# Link render element #} {{ link('Home', '<front>', {'class': ['home-link']}) }} {# Create_attribute — build dynamic attribute objects #} {% set attrs = create_attribute({'class': ['card'], 'id': 'card-1'}) %} <div {{ attrs }}>...</div> {# Using the built-in attributes variable #} <article {{ attributes.addClass('custom-class') }}> <article {{ attributes.setAttribute('data-id', node.id()) }}>

Global variables available in all templates

Variable	        Description
{{ is_front }}	    Boolean — true on the front page
{{ is_admin }}	    Boolean — true on admin pages
{{ logged_in }}	    Boolean — true if user is logged in
{{ language }}	    Current language object
{{ user }}	        Current user account object
{{ directory }}	    Theme's directory path (for assets)
{{ base_path }}	    Drupal base path (usually /)
{{ db_offline }}	True if site is in maintenance mode

Practical patterns with Drupal globals
{# Conditional for front page #} {% if is_front %} <h1>{{ site_name }}</h1> {% else %} <a href="{{ path('<front>') }}"><span>{{ site_name }}</span></a> {% endif %} {# Asset path using directory variable #} <img src="{{ base_path ~ directory }}/images/logo.svg" alt="Logo"> {# Admin-only debug block #} {% if is_admin %} <pre>{{ node|json_encode(constant('JSON_PRETTY_PRINT')) }}</pre> {% endif %}


## Template suggestions

Drupal builds a list of template candidates in specificity order — most specific first. Your theme can override any template without touching core.
Enable Twig debug in services.yml to see all template suggestions in your browser's HTML comments. Set twig.config: debug: true.

Node templates — specificity order
Template file	Matches
node--123--full.html.twig	Node ID 123, full view mode
node--123.html.twig	Node ID 123, any view mode
node--article--full.html.twig	Article type, full view mode
node--article.html.twig	Any article node
node--full.html.twig	Any node, full view mode
node.html.twig	Any node (fallback)


Q. Difference between .html.twig and .twig files
