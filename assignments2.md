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
