Q1. What is the difference between hooks and preprocesses?
Q2. What is the difference between html.html.twig and page.html.twig?
Q3. What is the difference between {{ }}, {% %} and {# #} in Twig — when do you use each?
Q4. What does the pipe | operator do in Twig? Explain chaining filters with a real example.

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
