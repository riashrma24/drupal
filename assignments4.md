for 4.2

1. in sky.theme

.
.
.
/**
 * Implements hook_preprocess_HOOK() for page.html.twig.
 */
function sky_preprocess_page(array &$variables): void {
  $variables['#attached']['library'][] = 'sky/sticky-header';
  $variables['#attached']['drupalSettings']['sky']['stickyThreshold'] = 100;
}

note : A sentinel is a zero-height div that the IntersectionObserver watches. When it scrolls out of view, the header becomes sticky.

2. in page.html.twig add sentinel element at top

<div id="page-wrapper" class="page-wrapper">
  <div id="page">

      <div class="sticky-sentinel" aria-hidden="true"></div>

3. create sticky-header.js

/**
 * @file
 * Sticky header using IntersectionObserver + drupalSettings.
 */
(function (Drupal, once, drupalSettings) {

    'use strict';

    Drupal.behaviors.stickyHeader = {
        attach(context, settings) {

            once('sticky-header', 'body', context).forEach(function () {

                const header = document.querySelector('.site-header');
                const sentinel = document.querySelector('.sticky-sentinel');

                if (!header || !sentinel) return;
                const threshold = settings.sky?.stickyThreshold ?? 100;

                const observer = new IntersectionObserver(
                    function (entries) {
                        entries.forEach(function (entry) {
                            if (entry.isIntersecting) {
                                header.classList.remove('is-sticky');
                            } else {
                                header.classList.add('is-sticky');
                            }
                        });
                    },
                    {
                        rootMargin: `${threshold}px 0px 0px 0px`,
                    }
                );

                observer.observe(sentinel);
            });

        }
    };

}(Drupal, once, drupalSettings));


4. append the following to header.css

.sticky-sentinel {
  height: 0;
  pointer-events: none;
}

.site-header.is-sticky {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 200;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  animation: slideDown 0.2s ease;
}

@keyframes slideDown {
  from { transform: translateY(-100%); }
  to   { transform: translateY(0); }
}


5. register library in sky.libraries.yml

sticky-header:
  js:
    js/sticky-header.js: {}
  css:
    component:
      css/component/header.css: {}
  dependencies:
    - core/drupal
    - core/once
    - core/drupalSettings

6. run ddev drush cr

