for 2.1
1. create a file

themes/custom/YOUR_THEME/
└── templates/
    └── node/
        └── node--article--teaser.html.twig

2. put the following code in it

   {#
/**
 * @file
 * node--article--teaser.html.twig
 *
 * Template for Article nodes displayed in "Teaser" view mode.
 *
 * Available variables:
 * - node: The node entity.
 * - content: Render array of node fields.
 * - url: Direct URL to the full node.
 * - label: Node title.
 */
#}

{{ attach_library('sky/article-card') }}
{%
  set classes = [
    'node',
    'node--type-' ~ node.bundle|clean_class,
    'node--view-mode-' ~ view_mode|clean_class,
    'article-teaser',
  ]
%}

<article{{ attributes.addClass(classes) }}>

  {# ── 1. FEATURED IMAGE ─────────────────────────────────────── #}
  {% if content.field_image %}
    <div class="article-teaser__image">
      <a href="{{ url }}" tabindex="-1" aria-hidden="true">
        {{ content.field_image }}
      </a>
    </div>
  {% endif %}

  <div class="article-teaser__body">

    {# ── 2. CATEGORY TAGS ───────────────────────────────────────── #}
    {% if node.field_tags is not empty %}
      <div class="article-teaser__tags">
        {% for tag in node.field_tags %}
          <a href="{{ path('entity.taxonomy_term.canonical', {'taxonomy_term': tag.target_id}) }}"
             class="article-teaser__tag">
            {{ tag.entity.name.value }}
          </a>
        {% endfor %}
      </div>
    {% endif %}

    {# ── 3. TITLE (linked) ──────────────────────────────────────── #}
    <h3 class="article-teaser__title">
      <a href="{{ url }}" rel="bookmark">{{ label }}</a>
    </h3>

    {# ── 4. BODY EXCERPT (160 chars, no HTML) ───────────────────── #}
    {% set excerpt = content.body|render|striptags|trim %}
    {% if excerpt %}
      <p class="article-teaser__excerpt">
        {{ excerpt|slice(0, 160) }}{% if excerpt|length > 160 %}…{% endif %}
      </p>
    {% endif %}

    {# ── 5. META: Author + Date ─────────────────────────────────── #}
    <footer class="article-teaser__meta">

      {# Author name #}
      {% if node.getOwner() %}
        <span class="article-teaser__author">
          By {{ node.getOwner().getDisplayName() }}
        </span>
      {% endif %}

      {# Date formatted as "Jan 15, 2026" #}
      <time class="article-teaser__date"
            datetime="{{ node.created.value|format_date('html_date') }}">
        {{ node.created.value|format_date('custom', 'M j, Y') }}
      </time>

    </footer>

  </div>{# /.article-teaser__body #}

</article>

3. add this css in article-card.css at sky/css/component/article-teaser.css

   /* article-teaser.css */

.article-teaser {
  display: flex;
  flex-direction: column;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
  background: #fff;
}

.article-teaser__image img {
  width: 100%;
  height: 200px;
  object-fit: cover;
  display: block;
}

.article-teaser__body {
  padding: 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

/* Tags */
.article-teaser__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
}

.article-teaser__tag {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #0066cc;
  text-decoration: none;
  background: #e8f0fe;
  padding: 0.2rem 0.5rem;
  border-radius: 3px;
}

.article-teaser__tag:hover {
  background: #0066cc;
  color: #fff;
}

/* Title */
.article-teaser__title {
  margin: 0;
  font-size: 1.125rem;
  line-height: 1.4;
}

.article-teaser__title a {
  color: #1a1a1a;
  text-decoration: none;
}

.article-teaser__title a:hover {
  color: #0066cc;
}

/* Excerpt */
.article-teaser__excerpt {
  margin: 0;
  font-size: 0.9rem;
  color: #555;
  line-height: 1.6;
}

/* Meta */
.article-teaser__meta {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-top: auto;
  padding-top: 0.75rem;
  border-top: 1px solid #f0f0f0;
  font-size: 0.8rem;
  color: #888;
}

.article-teaser__author {
  font-weight: 500;
}

.article-teaser__date {
  margin-left: auto;
}

4. append the following in sky.libraries.yml

article-card:
  css:
    component:
      css/component/article-card.css: {}

5. ddev drush cr
6. goto homepage, the articles' teaser view will be displayed there

for 2.2

1. create a file themes/custom/YOUR_THEME/templates/node/node--article--full.html.twig

{{ attach_library('sky/article-hero') }}
{#
/**
 * @file
 * node--article--full.html.twig
 *
 * Conditional Hero with 3-level fallback:
 *   1. field_hero_image (if attached to node)
 *   2. Theme's default fallback image asset
 *   3. CSS gradient (no image at all)
 */
#}

{% if node.field_image.0.entity.id %}
  {% set hero_url = file_url(node.field_image.0.entity.uri.value) %}
  {% set hero_type = 'image' %}
{% else %}
  {% set hero_url = null %}
  {% set hero_type = 'gradient' %}
{% endif %}

<section class="hero hero--{{ hero_type }}"{% if hero_type == 'image' %} style="background-image: url('{{ hero_url }}'); background-size: cover; background-position: center;"{% endif %}>
  <div class="hero__overlay"></div>
  <div class="hero__content">
    {% if node.field_tags is not empty %}
      <div class="hero__tags">
        {% for tag in node.field_tags %}
          <a href="{{ path('entity.taxonomy_term.canonical', {'taxonomy_term': tag.target_id}) }}"
             class="hero__tag">
            {{ tag.entity.name.value }}
          </a>
        {% endfor %}
      </div>
    {% endif %}
    <h1 class="hero__title">{{ label }}</h1>
    <div class="hero__meta">
      <span class="hero__author">By {{ node.getOwner().getDisplayName() }}</span>
      <span class="hero__date">
        {{ node.created.value|format_date('custom', 'M j, Y') }}
      </span>
    </div>
  </div>

</section>

<article class="article-full">
  <div class="article-full__body">
    {{ content.body }}
  </div>
</article>

2. Add a CSS file - /Users/sandeep/osl-projects/my-drupal-site/web/themes/sky/css/component/hero.css

.hero {
  position: relative;
  min-height: 420px;
  display: flex;
  align-items: flex-end;
}

.hero--gradient {
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
}

.hero__overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
}

.hero__content {
  position: relative;
  z-index: 1;
  padding: 2.5rem;
  color: #fff;
  width: 100%;
}

.hero__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-bottom: 0.75rem;
}

.hero__tag {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #fff;
  background: rgba(255, 255, 255, 0.2);
  padding: 0.2rem 0.6rem;
  border-radius: 3px;
  text-decoration: none;
}

.hero__tag:hover {
  background: rgba(255, 255, 255, 0.35);
}

.hero__title {
  margin: 0 0 0.75rem;
  font-size: 2.25rem;
  line-height: 1.2;
  color: #fff;
}

.hero__meta {
  display: flex;
  gap: 1.5rem;
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.8);
}

.article-full {
  max-width: 820px;
  margin: 0 auto;
  padding: 2.5rem 1.5rem;
}

3. Attach this css to the twig file

{{ attach_library('sky/article-hero') }}

4. Define its library in .libraries.yml

article-hero:
  css:
    component:
      css/component/hero.css: {}
