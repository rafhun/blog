---
title: Using colors in dynamic blocks
date: 2020-11-06T20:00
---

It is as easy as declaring support for the color picker for your blocks to add a color settings panel for your blocks. We have already seen how to handle these in normal blocks (if no customization is necessary you do not have to do anything). Let's no look at the helper we need to generate the necessary classes and possibly inline styles in dynamic blocks.

## Supports

For clarity here is what you add to the supports property in your `block.json` file.

```json
{
  "supports": {
    "__experimentalColor": true
  }
}
```

This example just deals with the default text and background colors. However you can use the same approach if you additionally declare support for gradients and/or the link color.

## PHP Helper

This example demonstrates a possible class based helper setup.

```php
<?php
namespace MyPlugin\Helpers;

class CustomColors
{
    /**
     * Generates classes and styles for possible custom text and background colors
     *
     * @param array $attributes - The block attributes
     *
     * @return array $css - Contains keys text and background with class and inline style information
     */
    public static function generateStyles(array $attributes)
    {
        $textColors = self::textColor($attributes);
        $backgroundColors = self::backgroundColor($attributes);

        return [
            'text' => $textColors,
            'background' => $backgroundColors,
        ];
    }
}
```

This is the main setup of our `CustomColors` helper class. It contains a static method used to generate an array of classes and possibly styles we can work with. It expects to receive the block attributes array as its sole argument. This method's only concern is to then compile the returns of two helper functions, put them into an array and return it.

Let us look at what is actually happening within the helpers.

```php
<?php
namespace MyPlugin\Helpers;

class CustomColors
{
    // ...

    /**
     * Generate classes and possibly inline styles if there are any custom text colors set
     *
     * @param array $attributes - The block attributes
     *
     * @return array $textColors - Array with an array of css classes and a string of inline styles
     */
    private static function textColor(array $attributes)
    {
        $textColors = [
            'classes' => [],
            'inlineStyles' => '',
        ];

        $hasNamedTextColor = array_key_exists('textColor', $attributes);
        $hasCustomTextColor = array_key_exists('customTextColor', $attributes);

        // If there is a text color
        if ($hasCustomTextColor || $hasNamedTextColor) {
            // Add has-text-color class
            $textColors['classes'][] = 'has-text-color';
        }

        if ($hasNamedTextColor) {
            // Add the color class.
            $textColors['classes'][] = sprintf(
                'has-%s-color',
                $attributes['textColor'],
            );
        } elseif ($hasCustomTextColor) {
            // Add the custom color inline style
            $textColors['inlineStyles'] .= sprintf(
                'color: %s;',
                $attributes['customTextColor'],
            );
        }

        return $textColors;
    }
}
```

This private method return an array in which possible custom classes are stored as an array and inline styles as a string. As its argument we just pass the `$attributes` of the block.

In a next step we check the attributes for a named and a custom text color. If either of those exist we can assign `has-text-color` the WordPress default text color indicator class to the classes array.

If there is a named text color assigned the attribute `textColor` contains that colors name. This is transformed into the correct class name (again respecting WordPress standards) using `sprintf`.

In the other case when there is a custom text color, we generate the corresponding inline style using `sprintf` once more. We then return the array we just created.

Now the process for the background color is the exact same, just the naming is slightly different. Check the full class example at the end of this post for reference.

## Usage in Render Function

To apply the classes and inline styles to your rendered component you can store the base information from the helper in a variable that you can then manipulate as follows. The below code assumes that the `Helpers` namespace is available through autoload or manual includes. Adjust as necessary for your project.

```php
<?php
// Inside of the render function
$colors = Helpers\CustomColors::generateStyles($attributes);
$classes = array_merge(
  $colors['background']['classes'],
  $colors['text']['classes'],
);
$classAttribute = sprintf(
  ' class="%s"',
  esc_attr(implode(' ', $classes)),
);
$styleAttribute =
  $colors['background']['inlineStyles'] ||
  $colors['text']['inlineStyles']
    ? sprintf(
      ' style="%s"',
      esc_attr($colors['background']['inlineStyles']) . esc_attr($colors['text']['inlineStyles']),
    )
    : '';
$blockContent = sprintf(
  '<div %1$s %2$s>',
  $classAttribute,
  $styleAttribute,
);

// Continue render
```

Classes are first merged together into one array. Here you also have an opportunity to add more classes, i. e. custom component classes or ones generated by bock alignment controls or other block controls. Just make sure to pass them within an array. The array is then imploded, escaped and put into a class attribute.

Possible inline styles are just empty strings if not set so we can check if either one exists and if so just concatenate them together, escape them and put them into a style attribute.

Then add the classes and styles to the appropriate container element of your block.

**Hint:** You can also separate background and text classes and styles if you want to put them on different elements, i. e. if you need more specificity for certain elements.

Check out the whole helper class example below if you want to see it all put together.

```php
<?php
namespace MyPlugin\Helpers;

class CustomColors
{
    /**
     * Generates classes and styles for possible custom text and background colors
     *
     * @param array $attributes - The block attributes
     *
     * @return array $css - Contains keys text and background with class and inline style information
     */
    public static function generateStyles(array $attributes)
    {
        $textColors = self::textColor($attributes);
        $backgroundColors = self::backgroundColor($attributes);

        return [
            'text' => $textColors,
            'background' => $backgroundColors,
        ];
    }

    /**
     * Generate classes and possibly inline styles if there are any custom text colors set
     *
     * @param array $attributes - The block attributes
     *
     * @return array $textColors - Array with an array of css classes and a string of inline styles
     */
    private static function textColor(array $attributes)
    {
        $textColors = [
            'classes' => [],
            'inlineStyles' => '',
        ];

        $hasNamedTextColor = array_key_exists('textColor', $attributes);
        $hasCustomTextColor = array_key_exists('customTextColor', $attributes);

        // If there is a text color
        if ($hasCustomTextColor || $hasNamedTextColor) {
            // Add has-text-color class
            $textColors['classes'][] = 'has-text-color';
        }

        if ($hasNamedTextColor) {
            // Add the color class.
            $textColors['classes'][] = sprintf(
                'has-%s-color',
                $attributes['textColor'],
            );
        } elseif ($hasCustomTextColor) {
            // Add the custom color inline style
            $textColors['inlineStyles'] .= sprintf(
                'color: %s;',
                $attributes['customTextColor'],
            );
        }

        return $textColors;
    }

    /**
     * Generate classes and possibly inline styles if there are any custom background colors set
     *
     * @param array $attributes - The block attributes
     *
     * @return array $backgroundColors - Array with an array of css classes and a string of inline styles
     */
    private static function backgroundColor(array $attributes)
    {
        $backgroundColors = [
            'classes' => [],
            'inlineStyles' => '',
        ];

        $hasNamedBackgroundColor = array_key_exists(
            'backgroundColor',
            $attributes,
        );
        $hasCustomBackgroundColor = array_key_exists(
            'customBackgroundColor',
            $attributes,
        );

        // If has background color.
        if ($hasCustomBackgroundColor || $hasNamedBackgroundColor) {
            // Add has-background-color class
            $backgroundColors['classes'][] = 'has-background';
        }

        if ($hasNamedBackgroundColor) {
            // Add the background-color class.
            $backgroundColors['classes'][] = sprintf(
                'has-%s-background-color',
                $attributes['backgroundColor'],
            );
        } elseif ($hasCustomBackgroundColor) {
            // Add the custom background-color inline style.
            $backgroundColors['inlineStyles'] .= sprintf(
                'background-color: %s;',
                $attributes['customBackgroundColor'],
            );
        }

        return $backgroundColors;
    }
}
```
