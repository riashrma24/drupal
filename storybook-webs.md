1.  mkdir my-drupal-site
2.  cd my-drupal-site
3.  ddev config --docroot=web --project-type=drupal11
4.  ddev start
5.  ddev composer create-project drupal/recommended-project .
6.  ddev composer require drush/drush
7.  ddev drush site:install --account-name=admin --account-pass=admin -y
8.  ddev launch
9.  ddev composer require drupal/pathauto
10. ddev composer require drupal/devel
11. ddev composer require drupal/admin_toolbar
12. ddev drush en pathauto devel admin_toolbar admin_toolbar_tools -y
13. ddev drush generate theme
     → named it: plant
     → base theme selected: claro (WRONG — changed to stable9 later)
     → generated files: plant.info.yml, plant.libraries.yml, plant.theme,
                        css/, js/, images/, templates/, package.json

14. Opened web/themes/custom/plant/plant.info.yml
     → changed: base theme: claro
     → to:      base theme: stable9

15. Opened web/themes/custom/plant/plant.info.yml
     → added plant/tokens to libraries:
       libraries:
         - plant/global
         - plant/tokens

16. Opened web/themes/custom/plant/plant.libraries.yml
     → added at the bottom:
       tokens:
         css:
           theme:
             components/00-tokens/tokens.css: {}

17. Ran:
     ddev drush theme:enable plant -y
     ddev drush config:set system.theme default plant -y
     ddev drush cr
     ddev launch

18. Created folder structure inside web/themes/custom/plant/:
     mkdir -p components/00-tokens
     mkdir -p components/01-atoms/button
     mkdir -p components/02-molecules/header
     mkdir -p components/02-molecules/testimonial-card
     mkdir -p components/03-organisms/hero-section
     mkdir -p components/03-organisms/services-section
     mkdir -p components/03-organisms/why-choose-section

19. Created web/themes/custom/plant/components/00-tokens/tokens.css
     → added all CSS custom properties (colors, typography, spacing, etc.)

20. cd web/themes/custom/plant
21. npm install
22. npx storybook@latest init --type html
     (dropped --no-open flag — it was not supported)

23. npm install --save-dev js-yaml-loader twig twig-loader

24. Opened web/themes/custom/plant/.storybook/main.js
     → replaced everything with custom config:
       - stories glob: components/**/*.stories.js
       - twig-loader wired with plant namespace
       - js-yaml-loader for .yml fixture files
       - framework: @storybook/html-webpack5

25. Opened web/themes/custom/plant/.storybook/preview.js
     → replaced everything with:
       - imports tokens.css
       - sets sage/lime/forest/white backgrounds
       - layout: fullscreen

26. Hit npm install errors — fixed package.json:
     → downgraded storybook to ^8.6.14 (from 10)
     → pinned all @storybook/* packages to ^8.6.14
     → fixed twig-loader version from ^0.6.2 to ^0.5.5
       (latest available: 0.5.5)
     → deleted node_modules + package-lock.json
     → ran npm install again

27. npm run storybook
     → Storybook 8.6.18 running at http://localhost:6007 ✓



    


Note — where we went wrong and why
1. Base theme was claro — drush generate theme defaults to claro which is Drupal's admin theme. Frontend themes should use stable9 (minimal, no opinions). We caught this early and fixed it manually in plant.info.yml.
2. --no-open flag on storybook init — this flag was removed in newer versions of Storybook. The fix was simply dropping it.
3. Storybook version mismatch — running npx storybook@latest init installed Storybook 10, but @storybook/html-webpack5 only supports Storybook 8. They use incompatible peer dependencies and npm refused to install them together. The fix was pinning everything to ^8.6.14 in package.json, deleting node_modules and reinstalling cleanly.
4. twig-loader version didn't exist — package.json had ^0.6.2 but the latest published version is 0.5.5. We ran npm info twig-loader versions to find the correct version and updated package.json accordingly.
