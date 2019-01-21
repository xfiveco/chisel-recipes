# Content Security Policy

_This recipe shows how to setup [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) in Chisel_

We added two new methods to `[your-theme]/Chisel/Security.php` class to increase security of the site. `addSecureHttpHeaders` method adds common HTTP secure headers. Use `addCsp` method to set Content Security Policy directives relevant to your site.

```php
<?php

namespace Chisel;

/**
 * Class Security
 * @package Chisel
 *
 * Default security settings for Chisel
 */
class Security {

  public function __construct() {
    remove_action( 'wp_head', 'wp_generator' );
    add_filter( 'the_generator', '__return_null' );
    add_filter( 'xmlrpc_enabled', '__return_false' );
    add_filter( 'script_loader_src', array( $this, 'removeSrcVersion' ) );
    add_filter( 'style_loader_src', array( $this, 'removeSrcVersion' ) );
    add_action( 'send_headers', array( $this, 'addSecureHttpHeaders' ), 10, 0 );
    add_action( 'send_headers', array( $this, 'addCsp' ), 10, 0 );
  }

  public function removeSrcVersion( $src ) {
    global $wp_version;
    $version_str = '?ver=' . $wp_version;
    $offset = strlen( $src ) - strlen( $version_str );
    if ( $offset >= 0 && strpos( $src, $version_str, $offset ) !== false ) {
      return substr( $src, 0, $offset );
    }

    return $src;
  }

  /**
   * Add common secure HTTP headers
   */
  public function addSecureHttpHeaders() {
    header("X-Frame-Options: SAMEORIGIN");
    header("X-XSS-Protection: 1; mode=block");
    header("X-Content-Type-Options: nosniff");
  }

  /**
   * Add Content Security Policy
   *
   */
  public function addCsp() {
    $scripts = array();

    // Allow inline scripts in dev environment to make Browsersync work
    if ( defined( 'CHISEL_DEV_ENV' ) ) {
      $scripts[] = "'unsafe-inline'";
    // Add inline scripts hash - check out browser console for the hash of failing script or style
    } else {
      $scripts[] = "'sha256-usaNaDjYaJk13pKJ3+ZScs4bxEEsUZ+JOE61TTGapMY=' "; // no-js removal script
    }

    $csp = "default-src 'none';
            base-uri 'self';
            connect-src 'self';
            img-src 'self';
            style-src 'self';
            font-src 'self';
            frame-src 'self' www.youtube-nocookie.com www.youtube.com;
            frame-ancestors 'none';
            object-src 'none';
            form-action 'self';
            script-src 'self' " . implode( ' ', $scripts ) . ";";

    header("Content-Security-Policy: " . trim( preg_replace( '/\s+/', ' ', $csp ) ) );
  }
}
```
