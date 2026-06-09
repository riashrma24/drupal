mkdir my-drupal-site
cd my-drupal-site
ddev config --docroot=web --project-type=drupal11
ddev start
ddev composer create-project drupal/recommended-project .
ddev composer require drush/drush
ddev drush site:install --account-name=admin --pass=admin -y
ddev launch

----

ddev composer require drupal/pathauto
ddev composer require drupal/devel
ddev composer require drupal/admin_toolbar
ddev drush en pathauto devel admin_toolbar admin_toolbar_tools -y



for 1.1
1. ran ddev drush generate theme
2. .info.yml me add kiya region
3. page.html.twig copy kiya from olivero
4. page.html.twig - comment me region add kiya,
5. if conditional ke andar render add kara

for 1.2
1. 
