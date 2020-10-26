---
title: Gutenberg Gotchas
date: 2020-10-26T13:30
---

This post will be updated regularly and contains quick little tips and tricks when working with the WordPress block editor (Gutenberg).

## Get the current title

Use the following hook to get the current title of the post you are editing. Use this to i. e. prepopulate templates, link editor blocks to the title or generate some strings based on the title.

Hint: since this makes use of a hook make sure to place this within a functional component.

```js
import { useSelect } from "@wordpress/data"

const { postTitle } = useSelect(select => {
  const { getEditedPostAttribute } = select("core/editor")
  return { postTitle: getEditedPostAttribute("title") }
})
```

I prefer to return an object and destructure it as is done above since this let's us easily extract more data using the `useSelect` hook.

## Get Current Post Data

Make use of `getCurrentPost` to access the latest saved data about a post. This gives you access to a variety of data, it is what is returned by a REST query. Use this as follows.

```js
import { useSelect } from "@wordpress/data"

const { post } = useSelect(select => {
  const { getCurrentPost } = select("core/editor")

  return { post: getCurrentPost() }
})
```

You now can read the post data inside of the `post` variable. It is probably easiest to further destructure this object as in the following example.

```js
const {
  author,
  categories,
  content,
  date_gmt,
  featured_media,
  id,
  link,
  title,
  type,
} = post

// For custom post types: rename keys with hyphens during destructuring:
const {
  "rh-custom-taxonomy": customTaxonomy,
  "rh-custom-taxonomy-2": customTaxonomy2,
} = post
```

Taxonomy information is returned as an array of term ID's, the author and featured media also return just an ID, most other keys just contain strings.
