---
title: Branding the WordPress backend
date: 2020-10-26T21:35
---

Bring your client's brand into the WordPress admin and the login screen to further personalize the experience of site admins and editors.

## Options

You can hardcode the options and styles into your plugin or theme or better offer them as customizer options. If you want to use the customizer make sure to register controls for the following variables:

- Page Background: sets the background color of the login page
- Brand Color: used for the primary button on the login page
- Text Color: default text color on the login page
- Logo: Will be set as background image above the login form and in the admin bar
- Header URL: The logo on the login screen links to this URL
- Header Text
- Admin Footer Text: This text appears in the footer of backend pages

## Login Screen

There are a few hooks available for customiztion of the login screen.

### Page Styles

Create the additional styles needed for the login page based on options retrieved from the Customizer, then enqueue them using the `login_enqueue_scripts` action. By echoing the return of its function the styles are injected into the login page.

```php
<?php
// Get options from customizer ...

// Page wide styles
$pageStyles = '
  .login {
    background-color: ' . $pageBackground . ';
    color: ' . $textColor . ';
  }
';
```

Using the `.login` class allows you to override the default WordPress sitewide styles. Next we will set up the styling we need for the logo section.

```php
<?php
// ...
$logoStyles = '
  #login h1 a {
    background-image: url(' . $logoPath . ');
  }

  .login form h1.admin-email__heading {
    color: ' . $textColor . ';
  }
';
```

Finally we need to adjust the submit button styles. Here we keep with WordPress default button styles and are just adjusting the colors in use.

```php
<?php
// ...
$buttonStyles = '
  .wp-core-ui form .button {
    background-color: ' . $pageBackground . ';
    color: ' . $textColor . ';
    border-color: ' . $textColor . ';
  }

  .wp-core-ui form .button:focus,
  .wp-core-ui form .button:hover {
    box-shadow: 0 0 0 1px ' . $textColor . ';
    background-color: ' . $pageBackground . ';
    border-color: ' . $textColor . ';
    color: ' . $textColor . ';
  }

  .wp-core-ui form .button-primary {
    background-color: ' . $brandColor . ';
    border-color: ' . $brandColor . ';
    color: ' . $pageBackground . ';
  }

  .wp-core-ui form .button-primary:focus,
  .wp-core-ui form .button-primary:hover {
    box-shadow: 0 0 0 1px ' . $pageBackground . ', 0 0 0 3px ' . $brandColor . ';
    background-color: ' . $brandColor . ';
    border-color: ' . $brandColor . ';
    color: ' . $pageBackground . ';
  }
';
```

Finally we put together all CSS and echo it as part of the hook.

```php
// ...
echo '
  <style type="text/css">' .
    $pageStyles .
    $logoStyles .
    $buttonStyles .
  '</style>
';
```

### Header URL and Text

The logo above the login form is linked to `wordpress.org` by default. There is also some hidden text in place that is used by screen readers while the browser only displays the logo which is set as background image (see above). There are hooks that allow adjustment of both the URL and text which can be used as follows.

```php
<?php
add_filter('login_headerurl', function () {
  // Get the URL from customizer -> $url

  // Validate and return the URL
  if (esc_url_raw($url) === $url) {
    return $url;
  }

  // Fallback default
  if (!$url || '' === $url) {
    return 'https://wordpress.org';
  }
});

add_filter('login_headertext', function () {
  // Get from customizer -> $text
  return $text;
})
```

## Admin Area

There are a few branding options available in the admin area of a WordPress website. It is possible to render your own logo in the admin bar and you may adjust the footer text.

### Admin Bar Logo

Use the `wp_before_admin_bar_render` hook to display your own logo in the admin bar. You set this up by injecting some custom CSS that overrides core defaults.

```php
<?php
add_action('wp_before_admin_bar_render', function () {
  // Get the admin bar logo from customizer -> $logoPath
  echo '
  <style type="text/css">
    #wpadminbar #wp-admin-bar-wp-logo > .ab-item .ab-icon::before {
      background-image: url(' . $logoPath . ');
      background-position: 0 0;
      color: rgba(0, 0, 0, 0);
    }

    #wpadminbar #wp-admin-bar-wp-logo:hover > .ab-item .ab-icon::before {
      background-position: 0 0;
    }
  </style>
  ';
});
```

### Admin Footer

The `admin_footer_text` filter just expects you to echo a string.

```php
<?php
// Get the footer text from customizer -> $text
add_filter('admin_footer_text', function () {
  echo $text;
});
```
