---
title: Sanitizer Functions For the WordPress Customizer
date: 2020-11-08T21:00
---

When adding your own fields to the WordPress customizer it is necessary to define sanitizing functions. The following is a collection of customizers you need for various types of inputs.

## Radio Box

### Sanitizer

The value of a radio box must be a slug and be one of the existing options.

```php
<?php
function sanitizeRadioBox($input, $setting)
{
  // Must be a slug
  $input = sanitize_key($input);

  // Get list of available options
  $options = $setting->manager->get_control($setting->id)->choices;

  // Return the input if it is valid, otherwise return a default value
  return (array_key_exists($input, $options) ? $input : $setting->default);
}
```

### Add Control

The control has a type of `radio` and provides an array of choices.

```php
<?php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Control Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'radio',
    'choices' => [
      'one' => esc_html__('Option One', 'textdomain'),
      'two' => esc_html__('Option Two', 'textdomain'),
    ]
  ]
);
```

## Checkbox

### Sanitizer

The checkbox simply returns true if checked and false otherwise.

```php
<?php
function sanitizeCheckbox($input)
{
  return (isset($input) ? true : false);
}
```

### Add Control

The control just defines a label and the type `checkbox`.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => __('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'checkbox'
  ]
);
```

## Select Options

### Sanitizer

This is pretty much the same as for radio options. It must be a slug and exist in the options.

```php
function sanitizeSelect($input, $setting)
{
  // Must be a slug
  $input = sanitize_key($input);

  // Get list of available options
  $options = $setting->manager->get_control($setting->id)->choices;

  // Return the input if it is valid, otherwise return a default value
  return (array_key_exists($input, $options) ? $input : $setting->default);
}
```

### Add Control

Define a type of `select` and provide an array with options. You can add an empty key to set up a "Select an Option" prompt.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'select',
    'choices' => [
      '' => esc_html__('Please select', 'textdomain'),
      'one' => esc_html__('Option one', 'textdomain'),
      'two' => esc_html__('Option two', 'textdomain'),
    ]
  ]
);
```

## Text Input and Textarea

### Sanitizer

For basic inputs where you want to allow simple text only you can simply pass the native `wp_filter_nohtml_kses()` function when defining the setting.

```php
$wp_customize->add_setting(
  $settingId,
  [
    'sanitize_callback' => 'wp_filter_nohtml_kses',
  ]
);
```

### Add Control

Set a type of `text` and a label.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'text',
  ]
);
```

## Email

### Sanitizer

Use the native `sanitize_email()` function when defining the setting.

```php
$wp_customize->add_setting(
  $settingId,
  [
    'sanitize_callback' => 'sanitize_email',
  ]
);
```

### Add Control

Set a type of `email` and a label.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'email',
  ]
);
```

## URL

### Sanitizer

Use the native `esc_url_raw()` function when defining the setting.

```php
$wp_customize->add_setting(
  $settingId,
  [
    'sanitize_callback' => 'esc_url_raw',
  ]
);
```

### Add Control

Set a type of `url` and a label.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'url',
  ]
);
```

## Number

### Sanitizer

Use the native `absint()` or `intval()` function when defining the setting.

```php
$wp_customize->add_setting(
  $settingId,
  [
    'sanitize_callback' => 'absint', // Converts value to a non-negative integer
    'sanitize_callback' => 'intval', // Converts value to an integer
  ]
);
```

### Add Control

Set a type of `number` and a label.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'number',
  ]
);
```

## Drop-down Pages

### Sanitizer

Use the native `absint()` function when defining the setting, since the input value is a page ID.

```php
$wp_customize->add_setting(
  $settingId,
  [
    'sanitize_callback' => 'absint', // Converts value to a non-negative integer
  ]
);
```

### Add Control

Set a type of `dropdown-pages` and a label.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'dropdown-pages',
  ]
);
```

## File Input

### Sanitizer

Define allowed file types, then get the file's type from its file name.

```php
function sanitizeFile($file, $setting)
{
  // Allowed types
  $allowed = [
    'jpg|jpeg|jpe' => 'image/jpeg',
    'gif' => 'image/gif',
    'png' => 'image/png',
  ];

  // Check type from file name
  $extension = wp_check_filetype($file, $allowed);

  // If file has valid mime return else return default
  return ($extension['ext'] ? $file, $setting->default);
}
```

### Add Control

You need to make use of the `WP_Customize_Upload_Control` class to setup a file input.

```php
$wp_customize->add_control(
  new WP_Customize_Upload_Control(
    $wp_customize,
    $settingId,
    [
      'label' => esc_html__('Label', 'textdomain'),
      'section' => $sectionId,
    ]
  )
);
```

## CSS

### Sanitizer

Use the native `wp_strip_all_tags` function when defining the setting.

```php
$wp_customize->add_setting(
  $settingId,
  [
    'sanitize_callback' => 'wp_strip_all_tags',
  ]
);
```

### Add Control

Set a type of `textarea` and a label.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'textarea',
  ]
);
```

## Color HEX Codes

### Sanitizer

Use the native `sanitize_hex_color` function when defining the setting. Also add a default color here.

```php
$wp_customize->add_setting(
  $settingId,
  [
    'default' => '#000000',
    'sanitize_callback' => 'sanitize_hex_color',
  ]
);
```

### Add Control

Use the `WP_Customize_Color_Control` class to set up a color picker.

```php
$wp_customize->add_control(
  new WP_Customize_Color_Control(
    $wp_customize,
    $settingId,
    [
      'label' => esc_html__('Label', 'textdomain'),
      'section' => $sectionId,
    ]
  )
);
```

## HTML

### Sanitizer

Use the native `wp_kses_post` function when defining the setting.

```php
$wp_customize->add_setting(
  $settingId,
  [
    'sanitize_callback' => 'wp_kses_post', // Keeps only HTML tags that are allowed in post content
  ]
);
```

Alternatively you can use the `wp_kses()` function and define the tags you want to allow yourself.

```php
function sanitizeHTML($input)
{
  $allowed = [
    'a' => [
        'href' => [],
        'title' => [],
    ],
    'br' => [],
    'em' => [],
    'strong' => [],
  ];

  return wp_kses($input, $allowed);
}
```

### Add Control

Set a type of `textarea` and a label.

```php
$wp_customize->add_control(
  $settingId,
  [
    'label' => esc_html__('Label', 'textdomain'),
    'section' => $sectionId,
    'type' => 'textarea',
  ]
);
```

## More information and documentation

Find more on native sanitizing functions in the [WordPress Codex](https://codex.wordpress.org/Validating_Sanitizing_and_Escaping_User_Data). Also take a look through `/wp-includes/formatting.php` to see all of the sanitization and escaping functions WordPress has to offer.
