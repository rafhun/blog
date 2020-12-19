---
title: Working with the WordPress customizer logo
date: 2020-11-14T20:30
---

The WordPress customizer offers an easy interface for users to set their own logos. However by default the customization options seem limited as WordPress automatically generates the markup for the logo linking it to the front page. Let's check out how we can implement the logo into our themes.

## Setup

To enable the custom logo in the customizer your theme must declare support for it in your theme. Within that declaration you can set some options for the logo. Making full use of the defaults you can just declare:

```php
add_theme_support('custom-logo');
```

You can also pass in an array of options.

```php
$options = [
  'height' => 100,
  'width' => 400,
  'flex-height' => true,
  'flex-width' => true,
  'header-text' => ['site-title', 'site-description'],
  'unlink-homepage-logo' => true,
];

add_theme_support('custom-logo', $options);
```

Check out the [WordPress Theme Handbook](https://developer.wordpress.org/themes/functionality/custom-logo/) for full documentation of the options.

## Default

If you are happy with the default markup that is generated by core you can use the `get_custom_logo()` function.

```php
// Probably in your Header template
if (has_custom_logo()) {
  the_custom_logo(); // or echo get_the_custom_logo();
}
```

The logo is always linked to home unless the theme supports removing the link on the front page. If linked this returns the following markup.

```html
<a href="/link/to/home" class="custom-logo-link" rel="home">
  <img
    src="/path/to/logo.jpg"
    alt="Site Title (or custom alt text)"
    class="custom-logo"
    loading="false"
  />
</a>
```

## Custom Classes

If you wish to change the attributes on the actual image tag that contains the logo you can make use of the `get_custom_logo_image_attributes` filter. It let's you edit the attributes array of the `wp_get_attachment_image` call which is used to generate the markup.

```php
add_filter('get_custom_logo_image_attributes', function ($attr) {
  $attr['class'] = 'your-own-class';
  return $attr;
});
```

If you also want to control the link attributes you need to filter the whole output with the `get_custom_logo` filter. Since you are already filtering in this case anyways you might want to include possible image tag edits in this function. We make use of PHP's `str_replace` to change the class and add an additional attribute to the link and also change the logo image's class in the following example.

```php
add_filter('get_custom_logo', function ($html) {
  // Change link class, add a title
  $html = str_replace(
    'class="custom-logo-link"',
    'class="your-logo-link-class" title="' . esc_attr__('A custom title', 'textdomain') . '"',
    $html
  );

  // Change logo image class
  $html = str_replace(
    'class="custom-logo"', // This assumes that this class has not been changed with `get_custom_logo_image_attributes`
    'class="c-logo"',
    $html
  );

  return $html;
});
```

## Inlining SVG Logos

In some cases it makes sense to inline a given SVG logo, i. e. for controlled responsive behavior or some animation. This can be achieved by creating a custom function that is used to add the logo into your templates.

```php
function customLogo()
{
  $logoId = get_theme_mod('custom_logo');
  $logoMime = get_post_mime_type($logoId);

  if ('image/svg+xml' === $logoMime) {
    $logoPath = get_attached_file($logoId);

    if (file_exists($logoPath)) {
      return '
        <a
          class="c-logo-link"
          href="' . home_url('/') . '"
        >
          <span class="c-logo">' .
            file_get_contents($logoPath) .
          '</span>
        </a>
      ';
    }
    return get_custom_logo();
  }

  return get_custom_logo();
}
```

The function gets the ID of the logo image from the theme mods and then checks its mime type. If it is not an SVG we just return `get_custom_logo()`. However if we are working with an SVG we get its path and return the file contents into our markup. Adjust markup and handling of errors to your needs. Then wherever you add the logo in your templates just call `customLogo()` instead of `get_custom_logo()`.