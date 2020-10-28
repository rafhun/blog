---
title: Gutenberg and Post Data
date: 2020-10-27T17:00
---

In the different data stores the block editor provides there is a lot of information stored that might be useful to display.

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

## Get taxonomy data

Use `getEntityRecord` to retrieve information for the returned taxonomy terms. Continuing the above example you can use the following code to access data for all terms in `customTaxonomy`.

```jsx
const customTaxonomyTermLinks = useSelect(
  select => {
    const { getEntityRecord } = select("core")
    let loaded = true
    const links = customTaxonomy?.map(termId => {
      const term = getEntityRecord("taxonomy", "rh-custom-taxonomy", termId)

      if (!term) {
        return (loaded = false)
      }

      return (
        <a key={termId} href={term.link}>
          {term.name}
        </a>
      )
    })

    return loaded && links
  },
  [customTaxonomy]
)
```

What is happening here? We again make use of the `useSelect` hook since we want to retrieve information from the `data/core` store. Inside the `links` variable is set up by mapping the array of taxonomy id's. For each id we get the entity record then put the return data directly into an anchor element (you could also just return the whole term object). The second paramter of `useSelect` specifies that this function should only be run if `customTaxonomy` changes.

This make sense if you want to build a link list anyways since this way you only need to loop over the terms once.

The easiest way to turn the returned data now stored in the `customTaxonomyTermLinks` variable, use the `reduce` method as follows:

```js
let termLinks =
  customTaxonomyTermLinks &&
  customTaxonomyTermLinks.length === 0
    ? __('No terms found', 'text-domain)
    : customTaxonomyTermLinks.reduce((prev, curr) => [prev, ", ", curr]))
```

This example returns all links comma separated.
