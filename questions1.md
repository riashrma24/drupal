Q1. What is the difference between hooks and preprocesses?
Q2. What is the difference between html.html.twig and page.html.twig?

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
