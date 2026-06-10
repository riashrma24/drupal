Q1. What is the difference between hooks and preprocesses?
Q2. What is the difference between html.html.twig and page.html.twig?
Q3. What is the difference between {{ }}, {% %} and {# #} in Twig — when do you use each?
Q4. What does the pipe | operator do in Twig? Explain chaining filters with a real example.
Q5. what is this content.body? what do we get these variables content and body from? and do we get different variables in different templates?
Q6. What is the difference between node.field_image and content.field_image in Drupal Twig — when should you use each?
Q7. What is the difference between set and include and embed in Drupal Twig — give an example of when to use each?

sol1 : Hooks
1. A hook is Drupal's way of letting your code plug into the core system without modifying core files.
2. Naming convention : [theme_or_module_name] + _ + [hook_name]
3. They fire at system events — login, save, send — not at render time.
4. Parameters vary per hook — could be $node, $account, &$message
5. No output — since it causes side effects or modifies passed-in data
6. eg : training_theme_form_alter(&$form, $form_state, $form_id), training_theme_theme_suggestions_node_alter(&$suggestions, $variables)
7. Hooks can be implemented in modules as well, Drupal collects every implementation of a hook and runs them all. The order they run in is controlled by module/theme weight.
8. However, for themes, only the active theme's hook will run, not even the parent one(if it is overridden)
9. Alter hooks are special, it passes pass the same variable by reference through every implementation in sequence. Each one sees what the previous one did. This is how Drupal lets multiple things cooperate on modifying the same data.

Preprocesses
1. A preprocess is a specific category of hook that fires just before a Twig template renders. Its only job is to prepare variables for Twig.
2. Naming convention : [theme_or_module_name] + _preprocess_ + [template_name]
3. The &$variables is always passed by reference — you modify it directly and Twig automatically receives whatever you put in it.
4. Parameter is always &$variables — the array Twig will receive
5. Variables become {{ var }} in the Twig template
6. eg : training_theme_preprocess_node(&$variables), training_theme_preprocess_html(&$variables), training_theme_preprocess_page(&$variables)

Sol2 : html.html.twig

1. the entire document shell <html>, <head>, <body> tags, meta tags, CSS links, JS scripts, body classes and attributes
2. YOU RARELY TOUCH THIS
3. eg :
<html {{ html_attributes }}>        ← lang, dir attributes
<head>
  {{ head_title }}                  ← <title> tag content
  {{ head }}                        ← meta tags, favicons
  {{ styles }}                      ← ALL css files Drupal loads
</head>
<body {{ attributes }}>             ← body classes go HERE
  {{ page_top }}                    ← admin toolbar injects here
  {{ page }}                        ← THIS is where page.html.twig goes
  {{ page_bottom }}
  {{ scripts }}                     ← ALL js files at bottom
</body>
</html>

page.html.twig

1. everything INSIDE <body>, your regions, header, footer, content, the layout of visible page sections
2. THIS IS WHERE YOU DO YOUR WORK
3. eg :
<header>
  {{ page.header }}
  {{ page.primary_menu }}
</header>

<main>
  {{ page.content }}
  {{ page.sidebar }}
</main>

<footer>
  {{ page.footer_col_1 }}
  ...
</footer>

NOTE : vimp
Anything on the <body> tag    →  hook_preprocess_html()
  └── body classes
  └── lang attribute changes
  └── head meta additions

Anything in page regions      →  hook_preprocess_page()
  └── passing variables to header
  └── detecting front page
  └── sidebar visibility logic

Anything in a specific node   →  hook_preprocess_node()
  └── reading time
  └── field manipulation
  └── custom node variables

Sol3 : {{ }}, {% %} and {# #} are delimiters. These are not Drupal-specific — they are core Twig syntax. But Drupal adds its own context to each one. 

1. {{ }} — Output / Print
   - Evaluates an expression and prints the result to the HTML.
2. {% %} — Logic / Control
   - Executes logic — never prints anything by itself. Used for control structures, variable assignment, includes, and any statement that controls the flow of the template.
3. {# #} — Comments
   - Twig comments. Completely removed from the final HTML output — unlike HTML comments which are visible in View Source.

Note : vimp
Even though attach_library() looks like it should be {% %} logic, it is actually a function that returns a value so it uses {{ }}:
{# This looks wrong but IS correct — attach_library returns a render array #}
{{ attach_library('training_theme/article-card') }}

Because when Drupal's render system processes a render array that has no #markup, #theme, or #type key — only #attached — it outputs an empty string to the HTML but still processes the attachments.

The returned render array looks like : 
[
  '#attached' => [
    'library' => [
      'training_theme/article-card',
    ],
  ],
]

attach_library() in Twig
        ↓
returns #attached render array
        ↓
Drupal render system collects it
        ↓
merges with ALL other #attached on the page
        ↓
injects CSS into {{ styles }} in <head>
injects JS into {{ scripts }} before </body>

Note : vimp
[
  '#attached' => [

    // CSS/JS libraries
    'library' => [
      'training_theme/article-card',
      'core/drupal',
    ],

    // Pass PHP data to JavaScript
    'drupalSettings' => [
      'myTheme' => [
        'apiUrl' => 'https://example.com/api',
        'debug'  => TRUE,
      ],
    ],

    // Inject arbitrary CSS (avoid in production)
    'html_head' => [
      [
        ['#tag' => 'style', '#value' => 'body { color: red; }'],
        'my-inline-style',
      ],
    ],

    // Add http headers
    'http_header' => [
      ['X-Custom-Header', 'value'],
    ],
  ],
]

attach_library() only fills the library key. But in preprocess you can fill all of them:

function training_theme_preprocess_node(&$variables) {
  $variables['#attached']['library'][] = 'training_theme/article-card';
  $variables['#attached']['drupalSettings']['myTheme']['nodeId'] = $variables['node']->id();
}

Sol4 : The pipe | passes the output of one thing as the input to the next thing. It is a chain of transformations — each step receives what the previous step produced and does something to it.
eg : value | filter1 | filter2 | filter3
Read it left to right: start with value, hand it to filter1, take that result and hand it to filter2, take that result and hand it to filter3. The final result is what gets printed.

eg : content.body | render | striptags | slice(0, 160)

in content.body we have :
// What content.body actually is — a render array
[
  '#theme'     => 'field',
  '#formatter' => 'text_default',
  '#items'     => FieldItemList Object,
  '#object'    => Node Object,
  '#field_name'=> 'body',
  0 => [
    '#type'   => 'processed_text',
    '#text'   => '<p>This is the <strong>body</strong> content of the article.</p>',
    '#format' => 'basic_html',
  ],
  '#attached' => [...],
  '#cache'    => [...],
]

This is a PHP render array — not a string. You cannot slice it, strip tags from it, or do any string operation on it directly. It's a nested PHP array with metadata.

step 1 : | render

Passes the render array through Drupal's render system. Executes all the field formatters, processes the text format, and converts the entire thing into an HTML string.

<!-- What comes out of |render -->
<div class="field field--name-body field--type-text-with-summary">
  <div class="field__item">
    <p>This is the <strong>body</strong> content of the article.</p>
  </div>
</div>

step 2 : | striptags

Strips every HTML tag from the string. PHP's strip_tags() under the hood.

<!-- Input to striptags -->
<div class="field field--name-body">
  <div class="field__item">
    <p>This is the <strong>body</strong> content of the article.</p>
  </div>
</div>

<!-- Output from striptags -->
This is the body content of the article.

step 3 — |slice(0, 160)

Takes a substring from position 0 to position 160. Works exactly like PHP's substr().

<!-- Input: full plain text string -->
"This is the body content of the article. Lorem ipsum dolor sit amet consectetur adipiscing elit sed do eiusmod tempor incididunt ut labore."

<!-- Output: first 160 characters -->
"This is the body content of the article. Lorem ipsum dolor sit amet consectetur adipiscing elit sed do eiusmod tempor incididunt ut lab"

sol5 : Each template has its own specific set of variables that Drupal prepares for it. The variables you get in node.html.twig are completely different from what you get in block.html.twig or page.html.twig.

<img width="718" height="346" alt="image" src="https://github.com/user-attachments/assets/fa88c3a4-c0b1-44fd-8a41-0dbef45b5957" />

<img width="790" height="303" alt="image" src="https://github.com/user-attachments/assets/42078ece-34b4-4e8a-a218-0337b6c75704" />

<img width="730" height="312" alt="image" src="https://github.com/user-attachments/assets/ec2478e4-d021-4535-a48f-ecea21e7484d" />

<img width="791" height="235" alt="image" src="https://github.com/user-attachments/assets/b4227099-b657-4b17-8db6-5472c794e3be" />

<img width="855" height="321" alt="image" src="https://github.com/user-attachments/assets/93f72581-e418-4ffd-9933-df22355e6cc1" />

{# Enable twig debug first, then: #}
{{ dump(content) }}    {# see everything in content #}
{{ dump(node) }}       {# see the raw entity #}
{{ dump(_context) }}   {# see ALL variables available #}

There are two ways to get the values : 
1. via node - raw entity values

{# Plain text value #}
{{ node.body.value }}

{# Processed/formatted value (with HTML) #}
{{ node.body.processed }}

{# Summary field #}
{{ node.body.summary }}

{# Image alt text #}
{{ node.field_image.alt }}

{# Image file URI #}
{{ node.field_image.entity.uri.value }}

{# First tag name #}
{{ node.field_tags[0].entity.name.value }}

{# Node title #}
{{ node.title.value }}

{# Author name #}
{{ node.uid.entity.name.value }}

2. via content

{# Instead of this #}
{{ dump(content) }}

{# Do this — dump only the field you care about #}
{{ dump(content.body) }}

{# Or go even deeper — get just the raw text value #}
{{ dump(node.body.value) }}
{{ dump(node.field_image.alt) }}
{{ dump(node.field_tags[0].entity.name.value) }}

{# Dump just the keys of content — see what fields exist #}
{{ dump(content|keys) }}

From where do these variables come from?

content is built by hook_preprocess_node() which Drupal core runs automatically before any node template renders. You never call it yourself — Drupal fires it and hands the result to Twig.

Note : vimp
When Drupal runs preprocess for a node template, it doesn't just run YOUR function. It runs a chain of preprocess functions in a specific order:
1. template_preprocess()              ← Drupal core — runs first, always
2. template_preprocess_node()         ← Drupal core — node specific
3. olivero_preprocess_node()          ← base theme (if it has one)
4. training_theme_preprocess_node()   ← YOUR function — runs last

Your function doesn't replace the chain — it appends to the end of it. By the time your function runs, content, node, label, url are already sitting in $variables.

Where each variable comes from ?
// STEP 1 — template_preprocess()
// drupal/core/includes/theme.inc
// Sets variables available in EVERY template regardless of type:
$variables['attributes']        = new Attribute();
$variables['title_attributes']  = new Attribute();
$variables['logged_in']         = $user->isAuthenticated();
$variables['is_admin']          = $user->hasPermission('access admin');

// STEP 2 — template_preprocess_node()
// drupal/core/modules/node/node.module
// Sets variables specific to node templates:
$variables['node']      = $node;
$variables['content']   = $build;      // ← content comes from HERE
$variables['label']     = $node->label();
$variables['url']       = $node->toUrl()->toString();
$variables['teaser']    = $view_mode === 'teaser';
$variables['page']      = $view_mode === 'full';
$variables['date']      = $node->get('created')->value;

// STEP 3 — your_theme_preprocess_node()
// YOUR training_theme.theme file
// $variables already has everything above
// You just ADD to it:
$variables['reading_time'] = 5;   // ← your custom variable

Proof : 
function training_theme_preprocess_node(&$variables) {
  // By the time YOUR code runs, dump what's already in $variables
  // These were set by template_preprocess_node() BEFORE you ran
  \Drupal::logger('training_theme')->debug(
    'Variables already set by core: @vars',
    ['@vars' => implode(', ', array_keys($variables))]
  );
}

Sol6 : Difference
node.field_image    {# raw entity field object — data only #}
content.field_image {# render array — already prepared for display #}

node is the entity object itself — the raw PHP object representing the node. When you access fields through node, you are traversing the entity's field API directly. You get field objects, typed data, and raw values.

{# Raw field item object #}
{{ node.field_image }}

{# The file URI — raw path like public://image.jpg #}
{{ node.field_image.entity.uri.value }}

{# The alt text stored on the field #}
{{ node.field_image.alt }}

{# The image entity's file ID #}
{{ node.field_image.target_id }}

{# The actual file entity it references #}
{{ node.field_image.entity.filename.value }}

You are reaching directly into the entity's data. No display settings. No image styles. No field formatters. Just raw stored values.

content is a render array built by Drupal's field formatter system. It has already gone through the display settings you configured at: Structure → Content types → Article → Manage display

{# Fully rendered field — uses the image style you configured in Manage Display #}
{{ content.field_image }}

{# Entire content render array — all fields at once #}
{{ content }}

eg : {# content.field_image — outputs full rendered HTML #}
{{ content.field_image }}

{# Outputs something like: #}
<div class="field field--name-field-image">
  <img src="/sites/default/files/styles/large/public/image.jpg"
       alt="My alt text"
       width="800" height="600" />
</div>

{# ─────────────────────────────────────────── #}

{# node.field_image — gives you raw data to work with yourself #}
{{ node.field_image.entity.uri.value }}
{# Outputs just: public://image.jpg #}

{{ node.field_image.alt }}
{# Outputs just: My alt text #}

<img width="798" height="490" alt="image" src="https://github.com/user-attachments/assets/03afedbf-2b5b-435c-9974-632f359af853" />

using both together : 
{# Check existence via node (fast, no rendering) #}
{# Render via content (respects image style config) #}

{% if node.field_image.target_id %}
  <div class="article-card__image">
    {{ content.field_image }}
  </div>
{% endif %}

Sol7 : Three different tools for three different jobs
1. {% set %} — Store a value in a variable
- set has nothing to do with including other files. It just assigns a value to a variable so you can reuse it without repeating yourself.
- set can also store entire blocks of HTML using the block form:
{% set card_meta %}
  <div class="meta">
    <span>{{ node.owner.displayname }}</span>
    <time>{{ node.created.value|format_date('custom', 'M j, Y') }}</time>
  </div>
{% endset %}

{# Now use it wherever needed #}
{{ card_meta }}

Note : vimp
The most important use of set in Drupal is avoiding double rendering. If you wrote content.body|render|striptags twice in a template, Drupal would render the body field twice — double the work. set stores the result once and reuses it.

{# WRONG — renders body field twice, expensive #}
{% if content.body|render|striptags|trim %}
  {{ content.body|render|striptags|trim|slice(0,160) }}
{% endif %}

{# RIGHT — renders once, reuses #}
{% set excerpt = content.body|render|striptags|trim %}
{% if excerpt %}
  {{ excerpt|slice(0, 160) }}
{% endif %}

2. {% include %}

include inserts another Twig file at that point in the output. The included file is a static partial — it receives variables but you cannot override sections of it. What you include is what you get.






