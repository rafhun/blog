---
title: Custom Blocks with a Color Picker in core v5.5
date: 2020-10-30T20:00
---

In most cases block colors can be enabled using supports flags. This works well, if the colors can be applied to the containing block and do not require any special treatment. You can use this as follows in the `block.json` files (here we have added additional support for gradients and to set a link color).

```json
{
  "supports": {
    "__experimentalColor": {
      "gradients": true,
      "linkColor": true
    }
  }
}
```

## Attributes

If using the `supports` flag and that implementation is working for you, i. e. you are happy with color definitions being applied to the block, you can remove all existing attributes concerning coloring (such as `backgroundColor`, `customBackgroundColor`, etc.).

When more control is necessary make sure to include the following attributes in your `block.json` file.

```json
{
  "style": {
    "type": "object"
  },
  "backgroundColor": {
    "type": "string"
  },
  "textColor": {
    "type": "string"
  },
  "gradient": {
    "type": "string"
  }
}
```

Of course `gradient` is only necessary if you select to support gradients. When migrating, remove the `customBackgroundColor` and `customTextColor` attributes. Those are being merged into the `style` attribute.

### Migration

To ensure full compatibility with your new implementation you need to add the following to the `deprecated.js` file and make sure to include it in your exported settings.

```js
// deprecated.js
const migrateCustomColors = attributes => {
  // Skip if there are no custom colors
  if (!attributes.customTextColor && !attributes.customBackgroundColor) {
    return attributes
  }

  const style = { color: {} }

  // Migrate text color
  if (attributes.customTextColor) {
    style.color.text = attributes.customTextColor
  }

  // Migrate background color
  if (attributes.customBackgroundColor) {
    style.color.background = attributes.customBackgroundColor
  }

  return {
    ...attributes,
    style,
  }
}

const deprecated = [
  {
    // Only relevant is shown, add possible supports and save function of deprecated implementation as well
    attributes: {}, // Include all of them as they were defined
    migrate: migrateCustomColors,
  },
]

export default deprecated
```

With this in place older implementations of your block will automatically be updated to this one.

## The `edit` function

### Using `supports`

If you are using the `supports` flag only, remove everything concerning the color picker (actual color picker component, contrast checker, etc. probably the whole panel) and change your main block to the `Block` component as demonstrated below.

```jsx
import { __experimentalBlock as Block } from "@wordpress/block-editor"

function Edit(props) {
  // Everything else

  return <Block.div>The block contents.</Block.div>
}
```

Adjust the HTML container according to the markup you need and that makes sense semantically.

### Manual control

To gain some more control over colors, i. e. where they get applied, when they should show up, etc. you can set things up in the `edit` function. However you also need to copy some core code until a proper integration is provided.

Copy and adapt the `ColorEdit` function from the `core/button` [block](https://github.com/WordPress/gutenberg/blob/wp/5.5/packages/block-library/src/button/color-edit.js). Additionally you will need the `getColorAndStyleProps` function which in a similar way can be copied from the `core/button` [block](https://github.com/WordPress/gutenberg/blob/wp/5.5/packages/block-library/src/button/color-props.js).

In your `edit.js` partial you need to import these two functions. Then call `useSelect` to load the color settings from the block editor and transform its return using `getColorAndStyleProps`. These props can the be used to assign classes and styles to editor components as shown below.

```jsx
// edit.js
import { useSelect } from "@wordpress/data"
import { RichText, __experimentalBlock as Block } from "@wordpress/block-editor"

import ColorEdit from "./ColorEdit"
import getColorAndStyleProps from "./getColorAndStyleProps"

const { colors } = useSelect(select => {
  return select("core/block-editor").getSettings()
}, [])
const colorProps = getColorAndStyleProps(attributes, colors, true)

return (
  <>
    <ColorEdit {...props} />
    <Block.div>
      <RichText
        className={colorProps.className}
        style={{ ...colorProps.style }}
      />
    </Block.div>
  </>
)
```

If you want to remove support for gradient backgrounds and have copied the `ColorEdit` component from the `core/button` block, delete the following lines within the `settings` prop of the `ColorPanel` (around line 240).

```
gradientValue,
onGradientChange: onChangeGradient,
```

## The `save` function

### Using `supports`

Similar as to the `edit` function you can remove everything relating to colors (`getColorClassName`, `className` and `style` if not used for other things).

### Manual control

Again import the `getColorAndStyleProps` function which can then be used to transform the relevant attributes into classes and/or style rules.

```jsx
import getColorAndStyleProps from "./getColorAndStyleProps"

export default function save({ attributes }) {
  const colorProps = getColorAndStyleProps(attributes)
  const classes = colorProps.className
  const style = { ...colorProps.style }

  // Do whatever else, then return the component
  return (
    <div className={classes} style={style}>
      Component contents.
    </div>
  )
}
```
