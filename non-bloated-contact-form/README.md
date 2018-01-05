# Simple, non-bloated contact form

_Create a simple, non-bloated and fully customizable contact form in WordPress._

First, install and configure a plugin for sending emails from WordPress, for example [Easy WP SMTP](https://wordpress.org/plugins/easy-wp-smtp/).

Now, create class with contact form in `[your-theme]/Chisel/ContactForm.php`:

```php
<?php

namespace Chisel;

/**
 * Class ContactForm
 * @package Chisel
 *
 * Simple non-bloated contact form
 */
class ContactForm {
  public function __construct() {
    add_shortcode( 'contactform', array( $this, 'addShortcode' ) );
  }

  private function createForm() {
    $context['server_uri'] = esc_url( $_SERVER['REQUEST_URI'] );
    $context['name_value'] = isset( $_POST["cf-name"] ) ? esc_attr( $_POST["cf-name"] ) : '';
    $context['email_value'] = isset( $_POST["cf-email"] ) ? esc_attr( $_POST["cf-email"] ) : '';
    $context['phone_value'] = isset( $_POST["cf-phone"] ) ? esc_attr( $_POST["cf-phone"] ) : '';
    $context['message_value'] = isset( $_POST["cf-message"] ) ? esc_attr( $_POST["cf-message"] ) : '';

    \Timber\Timber::render('components/contact-form.twig', $context );
  }

  private function createMessage() {
    $context['name'] = sanitize_text_field( $_POST["cf-name"] );
    $context['email'] = sanitize_email( $_POST["cf-email"] );
    $context['phone'] = sanitize_text_field( $_POST["cf-phone"] );
    $context['message'] = esc_textarea( $_POST["cf-message"] );

    return \Timber\Timber::compile('components/email.twig', $context );
  }

  private function deliverMail() {

    if ( isset( $_POST['cf-submitted'] ) ) {

      $to = get_option( 'admin_email' );
      $subject = "Contact Form: " . get_the_title();
      $message = $this->createMessage();

      // If antispam field was empty and email sent, redirect to Thank you page
      if ( empty( $_POST['cf-s'] ) && wp_mail( $to, $subject, $message ) ) {
        wp_redirect( get_permalink( 22 ) ); // Update to ID of your Thank you page
      } else {
        echo '<p>Sorry, there was an error.</p>';
      }
    }
  }

  public function addShortcode() {
    ob_start();
    $this->deliverMail();
    $this->createForm();

    return ob_get_clean();
  }
}
```

This class adds a contact form shortcode. It has methods for creating contact form from a Twig template and for delivering email, where email template is also stored as a Twig template. It also features a simple honeypot antispam protection.

The Twig template for the contact form looks like follows and is stored in `[your-theme]/templates/components/contact-form.twig`:

```twig
<form action="{{post.link}}" method="post" id="#form">
  <label for="cf-name">Name</label>
  <input type="text" id="cf-name" name="cf-name" pattern="^[\p{L}0-9\s]+$" value="{{name_value}}" placeholder="Your name" required="required">

  <label for="cf-email">Email</label>
  <input type="email" id="cf-email" name="cf-email" value="{{email_value}}" placeholder="name@example.org" required="required">

  <label for="cf-phone">Phone</label>
  <input type="tel" id="cf-phone" pattern="[0-9 ]+" name="cf-phone" value="{{phone_value}}" placeholder="" required="required">

  <label for="cf-message">Message</label>
  <textarea name="cf-message" id="cf-message" cols="30" rows="8" placeholder="Your message" required="required">{{message_value}}</textarea>

  <input type="text" name="cf-s" value="" class="u-hidden">

  <p class="c-form__note">* All fields are required</p>

  <div class="u-text-center">
    <input type="submit" name="cf-submitted" value="Submit">
  </div>
</form>
```

The email template is in `[your-theme]/templates/components/email.twig`:

```twig
Name: {{name}}
Email: {{email}}
Phone: {{phone}}
Message: {{message}}
```

Once you have all the files setup, instantiate the ContactForm class in `[your-theme]/functions.php`:

```php
if ( \Chisel\Helpers::isTimberActivated() ) {
  new \Chisel\Settings();
  new \Chisel\Security();
  new \Chisel\Performance();
  new \Chisel\Media();
  new \Chisel\Site();
  new \Chisel\ContactForm();
} else {
  \Chisel\Helpers::addTimberAdminNotice();
}
```

Now you can use the contact form shortcode directly in the posts or in PHP templates like so:

```php
<?php
/**
 * Template Name: Contact
 * Description: The template for the contact page
 *
 * @package chisel
 */

$context = \Timber\Timber::get_context();

$context['form'] = do_shortcode('[contactform]');

\Timber\Timber::render( array( 'template-contact.twig', 'page.twig' ), $context );
```
