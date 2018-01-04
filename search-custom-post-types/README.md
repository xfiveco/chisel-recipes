# Search in custom post types

_This recipe demonstrates how search can be expanded in Chisel so it searches in custom post types too._

Imagine you have a custom post type _testimonial_ defined like in [this recipe](https://github.com/xfiveco/chisel-recipes/tree/master/shortcodes). You want to search in testimonials too.

Create `[your-theme]/templates/components/search.twig` template for search form and include it in your theme:

```twig
<form action="{{site.url}}" role="search" method="get">
  <input type="text" name="s" value="" placeholder="Search">
  <input type="submit">
</form>
```

Create `[your-theme]/Chisel/Search.php` class

```php
<?php

namespace Chisel;

/**
 * Class Search
 * @package Chisel
 *
 * Allow to search in custom post types
 */
class Search {

  public function __construct() {
    add_action( 'pre_get_posts', array($this, 'customSearch') );
  }

  /**
   * Create custom search query
   */
  public function customSearch( $query ) {
    if ( !is_admin() && $query->is_main_query() ) {
      if ( $query->is_search ) {
        $query->set('post_type', array( 'post', 'page', 'testimonial' ) );
      }
    }
  }
}
```

Add post type name in the `customSearch` method.

Instantiate the Search class in the `[your-theme]/functions.php`:

```php
if ( \Chisel\Helpers::isTimberActivated() ) {
  new \Chisel\Settings();
  new \Chisel\Security();
  new \Chisel\Performance();
  new \Chisel\Media();
  new \Chisel\Site();
  new \Chisel\Search();
} else {
  \Chisel\Helpers::addTimberAdminNotice();
}
```
