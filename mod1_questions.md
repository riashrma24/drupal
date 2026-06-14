Q: What are the mandatory files required to create a custom Drupal theme?

Expected Answer: At a bare minimum, you only need one file: THEMENAME.info.yml. This file provides Drupal with metadata about the theme (name, type, description, core version compatibility, and regions). However, to make it functional, you will almost always include a THEMENAME.libraries.yml for CSS/JS, and a THEMENAME.theme file for PHP preprocess functions.

Q: How do you declare and override "Regions" in a custom Drupal theme?
Expected Answer: Regions are defined in the THEMENAME.info.yml file under the regions: key.

Crucial Detail: If you define any custom regions in your .info.yml file, Drupal will completely ignore its default core regions. You must explicitly re-declare standard regions (like content, header, footer) if you intend to use them.

* what if we have declared a base theme
* what if we havent declared a base theme

Q: What is a Sub-theme in Drupal, and how does it inherit assets from a Base theme?

Expected Answer: A sub-theme inherits all the templates, regions, and assets of a parent (base) theme (such as Stable9, Olivero, or Bootstrap). It is defined in the .info.yml file using the line base theme: PARENT_THEME_NAME. To override a template, you copy the template file from the base theme into your sub-theme directory and clear the cache.

Q: Explain the purpose of the .libraries.yml file. How do you attach a library globally vs. conditionally?
Expected Answer: In modern Drupal, CSS and JavaScript are not loaded globally by default; they are organized into asset libraries.

Globally: Attach it via the theme's .info.yml file under the libraries: key.

Conditionally: Use the {{ attach_library('THEMENAME/library-name') }} function directly inside a specific Twig template, or inject it via a PHP preprocess function using $variables['#attached']['library'][] = 'THEMENAME/library-name';.

Q: What is the theme template suggestion hierarchy? How can you find out which template is currently rendering a component?

Expected Answer: Drupal uses a specific naming convention (hook-based) going from specific to generic (e.g., node--id--1.html.twig $\rightarrow$ node--article.html.twig $\rightarrow$ node.html.twig).To debug this, you enable Twig Development Mode in development.services.yml (setting twig.config: debug: true). This outputs HTML comments in the browser's Page Source showing all possible template suggestions and exactly which template file is currently in use.

Q: What is a Preprocess function, and how do you use it to pass data to Twig?

Expected Answer: Preprocess functions are PHP functions written in the THEMENAME.theme file. They intercept data before it is handed over to the Twig engine. They follow the naming convention THEMENAME_preprocess_HOOK(&$variables). For example, THEMENAME_preprocess_node can be used to alter node data or add a custom text variable that can later be printed via {{ custom_text }} in node.html.tw

Q: What are Render Arrays, and why are they critical to the Drupal theme architecture?

Expected Answer: Render arrays are structured, hierarchical PHP associative arrays used by Drupal to represent page content and layout before rendering it into HTML. They use properties starting with an # (e.g., #theme, #type, #markup, #cache). They are critical because they allow modules and themes to alter content programmatically (via hooks or subscribers) prior to rendering, preventing premature string building and ensuring robust caching.

Q: How does Drupal caching (Cache Tags, Contexts, and Max-Age) integrate with the theme layer?

Expected Answer: Caching metadata is passed through the theme layer inside render arrays:

Cache Tags (#cache']['tags']): Identify what data is being rendered (e.g., node:5). If node 5 changes, the cache is invalidated globally.

Cache Contexts (#cache']['contexts']): Define variations of the output (e.g., user.roles or languages).

Max-Age (#cache']['max-age']): Time-to-live in seconds.
If a front-end developer strips out or ignores render arrays by rendering raw values (like {{ content.body.0 }} instead of {{ content.body }}), they risk breaking the cache bubble-up system, causing stale content or rendering performance bottlenecks.

Q: How would you approach building a Component-Based Theme in Drupal?

Expected Answer: Modern best practices favor decoupling or componentizing the front end using methodologies like BEM (Block, Element, Modifier) and tools like Storybook or Pattern Lab. In Drupal, this is achieved using modules like Components (allowing Twig namespaces) or native Single Directory Components (SDC) introduced in core. You map Drupal's fields and view modes in theme templates to atomic design components using Twig statements like {% include %}, {% embed %}, or {% extend %}.

Q: What is the difference between {{ content|render }} and {{ content }} in a Twig template? Why does it matter?

Expected Answer: * {{ content }} passes the render array to the Twig engine, which renders it automatically, allowing cache metadata to bubble up properly to the page level.

{{ content|render }} explicitly forces the render array to become an HTML string right at that moment in execution.
Using the |render filter can break the caching hierarchy if not done carefully, because the cache metadata might not bubble up appropriately, leading to dynamic parts of the page being cached permanently or vice-versa.

Q: "A client wants a specific paragraph type to look completely different depending on whether it is placed on a 'Landing Page' or an 'Article' content type. How would you architect this in the theme layer?"

Expected Answer: This should be handled using template suggestions based on the parent entity. In the .theme file, implement hook_theme_suggestions_paragraph_alter(). Access the parent entity from the paragraph object, check its bundle type (landing_page or article), and append that as a new suggestion (e.g., paragraph__PARAGRAPH_TYPE__parent__LANDING_PAGE). This keeps the backend architecture clean without creating redundant paragraph fields or content types.


