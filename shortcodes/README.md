# Shortcodes in Chisel

_This recipe demonstrates how you can use shortcodes in Chisel._

Suppose you have a testimonial custom post type defined in `[your-theme]/Chisel/Extensions/DataType.php`:

```php
  /**
   * Use this method to register custom post types
   */
  public function registerPostTypes() {
    register_post_type( 'testimonial',
      array(
        'labels' => array(
          'name' => __( 'Testimonials' ),
          'singular_name' => __( 'Testimonial' ),
          'all_items' => __( 'All testimonials' ),
          'add_new' => __( 'Add testimonial' ),
          'add_new_item' => __( 'Add new testimonial' ),
          'edit_item' => __( 'Edit testimonial' ),
        ),
        'capability_type' => 'post',
        'exclude_from_search' => true,
        'has_archive' => false,
        'public' => false,
        'rewrite' => false,
        'show_in_nav_menus' => false,
        'show_in_menu' => true,
        'show_ui' => true,
        'supports' => array('title', 'editor')
      )
    );
  }
```

You can create a page with a list of all testimonials, but you can also insert a specific testimonial on certain pages or blog posts **using a shortcode**. The shorcode can look like `[testimonial id=4]`.

Create a new class `[your-theme]/Chisel/Testimonial.php`:

```php
<?php

namespace Chisel;

/**
 * Class Testimonial
 * @package Chisel
 *
 * Testimonial shortcode
 */
class Testimonial {
  public function __construct() {
    add_shortcode( 'testimonial', array( $this, 'testimonialShortcode' ) );
  }

  public function testimonialShortcode( $atts ) {
    if ( isset( $atts['id'] ) ) {
      $id = sanitize_text_field( $atts['id'] );
      $testimonial = \Timber\Timber::get_post( array( 'p' => $id, 'post_type' => 'testimonial' ) );
    } else {
      $testimonial = false;
    }

    return \Timber\Timber::compile( 'components/testimonial.twig', array( 'testimonial' => $testimonial ));
  }
}
```

First, we define shortcode, then we get testimonial post in the `testimonialShortcode` method. Next, we pass testimonial content to a Twig template and get an output using [Timber compile method](https://timber.github.io/docs/reference/timber/#compile).

The Twig template in `[your-theme]/templates/components/testimonial.twig` can look like this:

```twig
{% if testimonial %}
  <blockquote class="c-testimonial">
      <div class="c-testimonial__text">
        {{testimonial.content}}
      </div>
      <p class="c-testimonial__author">{{testimonial.title}}</p>
  </blockquote>
{% endif %}
```

We can add custom styling to `src/styles/components/_testimonial.scss`:

```css
.c-testimonial {
  margin: 2em;
}

.c-testimonial__text {
  font-size: 1.25rem;
  position: relative;
}

.c-testimonial__author {
  font-size: 1.1rem;
  font-style: normal;
  margin-top: 0.5em;
  text-align: right;
}
```

Finally, let's instantiate the Testimonial class in the `[your-theme]/functions.php`:

```php
if ( \Chisel\Helpers::isTimberActivated() ) {
  new \Chisel\Settings();
  new \Chisel\Security();
  new \Chisel\Performance();
  new \Chisel\Media();
  new \Chisel\Site();
  new \Chisel\Testimonial();
} else {
  \Chisel\Helpers::addTimberAdminNotice();
}
```
