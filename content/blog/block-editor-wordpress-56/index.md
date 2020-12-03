---
title: Block editor and WordPress 5.6
date: 2020-12-03T15:00
---

WordPress v5.6 brings a lot of changes to the editor.

## Widgets

The block based widgets screen is out of experiments and now available by default. To opt out of this screen remove theme support for it.

```php
remove_theme_support('widgets-block-editor');
```

## Supports

Several components have moved on from experimental status. Also color supports now differentiates between background and text colors. The following supports are available from v5.6.

```js
supports: {
  color: {
    background: true,
    gradient: true,
    text: true,
  },
  fontSize: true,
  lineHeight: true,
}
```

Control the preset values using [editor-color-palette](https://developer.wordpress.org/block-editor/developers/themes/theme-support/#block-color-palettes), [editor-gradient-presets](https://developer.wordpress.org/block-editor/developers/themes/theme-support/#block-gradient-presets) and [editor-font-size](https://developer.wordpress.org/block-editor/developers/themes/theme-support/#block-font-sizes).

### Dynamic block support

Use the newly available function `get_block_wrapper_attributes()` to automatically populate the necessary attributes in your block container.

```php
function RenderCallback($attributes)
{
  $wrapperAtts = get_block_wrapper_attributes();
  $markup = 'some block markup';
  return sprintf(
    '<div %1$s>%2$s</div>',
    $wrapperAtts,
    $markup
  );
}
```

The optional parameter `$extra_attributes` can be used to add more classes and/or inline styles. Important: the parameter must be an array and only the keys `class` and `style` are supported. Any values for these keys will be added to the generated attributes separated by a space.

### Behind the scenes

Adding support for these things automatically adds some attributes to the block, unless you have declared them yourself. You can optionally use these to set default values.

```js
attributes: {
  backgroundColor: {
    type: 'string',
    default: 'value',
  },
  gradient: {
    type: 'string',
    default: 'value',
  },
  textColor: {
    type: 'string',
    default: 'value',
  },
  fontSize: {
    type: 'string',
    default: 'value',
  },
  style: {
    type: 'object',
    default: {
      color: {
        background: 'value',
        gradient: 'value',
        link: 'value',
        text: 'value',
      },
      typography: {
        fontSize: 'value',
        lineHeight: 'value',
      },
    },
  },
}
```

The attributes `backgroundColor`, `gradient`, `textColor` and `fontSize` hold preset values the user has selected. If a custom value is used instead it is stored in the corresponding key within the `style` attribute.

## Block API version 2

This enables blocks to render their own block wrapper element and is an opt-in feature. To use the API version 2 set the following in your block settings.

```js
registerBlockType(name, { apiVersion: 2 })
```

However to make use of v2 you must also adjust your edit and save functions which now must make use of `useBlockProps`.

```jsx
import { useBlockProps } from "@wordpress/block-editor"

function Edit({ attributes }) {
  const blockProps = useBlockProps({
    className: someClassName,
    style: { color: "blue" },
  })

  return <p {...blockProps}>{attributes.content}</p>
}
```

```jsx
import { useBlockProps } from "@wordpress/block-editor"

function Save({ attributes }) {
  const blockProps = useBlockProps.save({
    className: someClassName,
    style: { color: "blue" },
  })

  return <p {...blockProps}>{attributes.content}</p>
}
```

Make sure to set the plugins `Requires at least` header field to `5.6` if you make use of v2.

## Toolbar

Do not use `<Toolbar>` within another toolbar to group related controls. Use `<ToolbarGroup>` instead. If you see a deprecation warning about a nested toolbar, change it to a toolbar group.

Also for accessibility reasons do not use elements such as `<button />` or `<Dropdown>` menu as direct descendent of a toolbar. Use `ToolbarButton` for regular buttons or `ToolbarItem` for any other kind of control (i. e. dropdowns).

```jsx
<BlockControls>
  <ToolbarItem as="button" />
  <ToolbarButton />
  <ToolbarItem>
    {itemProps => <DropdownMenu toggleProps={itemProps} />}
  </ToolbarItem>
</BlockControls>
```
