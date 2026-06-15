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

---

for 4.3

1. Create web/themes/sky/sky.routing.yml

sky.favorite_toggle:
  path: '/sky/favorite/{node}'
  defaults:
    _controller: '\Drupal\sky\Controller\FavoriteController::toggle'
    _title: 'Favorite Toggle'
  requirements:
    _permission: 'access content'
    node: \d+
  methods: [POST]

2. Create the controller

<?php

namespace Drupal\sky\Controller;

use Drupal\Core\Controller\ControllerBase;
use Drupal\node\NodeInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;

class FavoriteController extends ControllerBase {

  public function toggle(NodeInterface $node, Request $request): JsonResponse {
    $session = $request->getSession();
    $favorites = $session->get('sky_favorites', []);
    $nid = $node->id();

    if (in_array($nid, $favorites)) {
      $favorites = array_values(array_filter($favorites, fn($id) => $id !== $nid));
      $favorited = FALSE;
    }
    else {
      $favorites[] = $nid;
      $favorited = TRUE;
    }

    $session->set('sky_favorites', $favorites);

    return new JsonResponse(['status' => 'ok', 'favorited' => $favorited]);
  }

}


3. add a toggle button in node--article--full.html.twig

{{ attach_library('sky/favorite-toggle') }}

<article class="article-full">
....
  <div class="article-full__body">
    {{ content.body }}
  </div>

  <button
    class="favorite-btn"
    data-nid="{{ node.id() }}"
    aria-label="Add to favorites"
    aria-pressed="false">
    <span class="favorite-btn__icon">♡</span>
    <span class="favorite-btn__label">Favorite</span>
    <span class="favorite-btn__spinner" aria-hidden="true"></span>
  </button>

</article>

4. create js/favourite-toggle.js

/**
 * @file
 * AJAX favorite toggle — fetch() with Drupal CSRF token.
 */
(function (Drupal, once) {

    'use strict';

    Drupal.behaviors.favoriteToggle = {
        attach(context, settings) {

            once('favorite-toggle', '.favorite-btn', context).forEach(function (button) {

                button.addEventListener('click', async function () {
                    const nid = button.dataset.nid;

                    button.classList.add('favorite-btn--loading');
                    button.disabled = true;

                    try {
                        const token = await fetch('/session/token').then(r => r.text());

                        const response = await fetch(`/sky/favorite/${nid}`, {
                            method: 'POST',
                            headers: {
                                'X-CSRF-Token': token,
                                'Content-Type': 'application/json',
                            },
                        });

                        if (!response.ok) throw new Error(`Server error: ${response.status}`);

                        const data = await response.json();
                        if (data.favorited) {
                            button.classList.add('favorite-btn--active');
                            button.setAttribute('aria-label', 'Remove from favorites');
                            button.setAttribute('aria-pressed', 'true');
                            button.querySelector('.favorite-btn__icon').textContent = '♥';
                        }
                        else {
                            button.classList.remove('favorite-btn--active');
                            button.setAttribute('aria-label', 'Add to favorites');
                            button.setAttribute('aria-pressed', 'false');
                            button.querySelector('.favorite-btn__icon').textContent = '♡';
                        }

                    }
                    catch (error) {
                        button.classList.add('favorite-btn--error');
                        setTimeout(() => button.classList.remove('favorite-btn--error'), 3000);
                    }
                    finally {
                        button.classList.remove('favorite-btn--loading');
                        button.disabled = false;
                    }
                });

            });

        }
    };

}(Drupal, once));

5. Create css/component/favorite-toggle.css

.favorite-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1.25rem;
  border: 2px solid var(--color-primary, #0057b8);
  background: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background 0.2s ease, color 0.2s ease;
  margin-top: 1.5rem;
}

.favorite-btn--active {
  background: var(--color-primary, #0057b8);
  color: white;
}

.favorite-btn--error {
  border-color: #c0392b;
  color: #c0392b;
}

/* Spinner — hidden by default, shown during loading */
.favorite-btn__spinner {
  display: none;
  width: 14px;
  height: 14px;
  border: 2px solid currentColor;
  border-top-color: transparent;
  border-radius: 50%;
  animation: fav-spin 0.6s linear infinite;
}

.favorite-btn--loading .favorite-btn__spinner {
  display: inline-block;
}

.favorite-btn--loading .favorite-btn__icon,
.favorite-btn--loading .favorite-btn__label {
  opacity: 0.4;
}

@keyframes fav-spin {
  to { transform: rotate(360deg); }
}


6. Register library in sky.libraries.yml

favorite-toggle:
  js:
    js/favorite-toggle.js: {}
  css:
    component:
      css/component/favorite-toggle.css: {}
  dependencies:
    - core/drupal
    - core/once

7. ddev drush cr



