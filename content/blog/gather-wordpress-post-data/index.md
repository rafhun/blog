---
title: Gather Post Data in WordPress
date: 2020-11-03T20:15
---

To decouple logic and markup from each other, let's define a helper class that is in charge of gathering all relevant post data.

## Basic Class Setup

The helper class should be initialized with a post ID that might be returned from a custom query, some custom fields or any other source. Let's store this ID in a private variable so we can access it from everywhere.

```php
namespace Helpers;

class PostData
{
  /**
     * Holds the ID of the current post, assigned in the constructor
     */
    private int $postId;

    /**
     * The constructor takes in the ID argument and if it seems valid, assigns it to the internal $postId variable.
     *
     * @param int $id
     */
    public function __construct(int $id)
    {
        if (is_int($id)) {
            $this->postId = $id;
        }
    }

    /**
     * Collect data about the job, optionally metas only
     *
     * @param boolean $detailed Collect all data about the job
     *
     * @return object Data object containing some data about the job
     */
    public function getData(bool $detailed = true)
    {
      if ('post-type-slug' !== get_post_type($this->postId)) {
        // Bail if the post has the wrong post type
        return false;
      }

      // Gather data
    }
}
```

As seen above the next step is a generalized `getData` function. This is where we collect everything we need for displaying the post.

Add the `$detailed` argument if you wish to differentiate which data is gathered dependent on the context. You could for example use the detailed data on a single view while only gathering more general data on archive pages.

## Gathering Simple Data

With simple data we mean the information we can fully access using basic WordPress functions and that does not need to be manipulated. Let's continue the above example.

```php
namespace Helpers;

class PostData
{
  // ...
  public function getData(bool $detailed = true)
  {
    // ...

    // Initialize the data object that stores all information
    $data = (object)[];

    $data->title = html_entity_decode(get_the_title($this->postId));
    $data->permalink = get_permalink($this->postId);
    $data->excerpt = get_the_excerpt($this->postId);

    // Continue data collection as necessary

    return $data;
  }
}
```

## Advanced Data

There is some data that you might need or want to manipulate before sending it to a template. Let's look at two useful examples.

### Date

To properly add a date to your post you need to generate two date formats, one for computers (used in the `datetime` attribute) and on formatted for humans.

The following method gets that data and makes it available in an object.

```php
namespace Helpers;

class PostData
{
  // ...
  /**
    * Return the information necessary to display a proper date.
    *
    * @return object Contains the date information as `attribute` and `text`
    */
  private function getDate()
  {
      $date = (object) [];
      $date->attribute = get_post_time('c', true, $this->postId);
      $date->text = get_the_date('', $this->postId);

      return $date;
  }
}
```

All we are doing here is using two core WordPress functions and storing its returns into an object. Call this in the `getData` method if you need date information.

```php
$data->date = $this->getDate();
```

### Taxonomy Terms

In order for this to work properly for any taxonomies (categories, tags, custom taxonomies) it is best to use a generalized helper method. Call it directly and provide the taxonomy slug as argument or set up a secondary helper method where the taxonmy slug is hardcoded.

```php
namespace Helpers;

class PostData
{
  // ...
  /**
    * Returns an array of objects containing the name and link of all terms of a specific taxonomy that are assigned to the current post.
    *
    * @param string $taxonomySlug
    *
    * @return array
    */
  private function getTermData(string $taxonomySlug)
  {
      $getTerms = get_the_terms($this->postId, $taxonomySlug);

      if (!$getTerms || is_wp_error($getTerms)) {
        return false;
      }

      $terms = [];

      foreach ($getTerms as $theTerm) {
          $term = (object) [];
          $term->name = html_entity_decode($theTerm->name);
          $term->link = get_term_link($theTerm);
          $terms[] = $term;
      }

      return $terms;
  }
}
```

What we are doing here is simply making use of the `get_the_terms` function providing our stored post ID and the given taxonomy slug. If any terms of the taxonomy are assigned to this post it will return an array of term objects. We return false if no terms could be found.

Next we initialize our own terms array which we then populate with reduced term objects. Loop through the returned terms using `foreach` then extract whatever information you need into the `$term` object. In this example we are getting the name and link to the archive of each term and then add the whole object to our `$terms` array.

If you want to call this helper indirectly add the following method.

```php
/**
  * Gets the terms assigned from the `taxonomy-slug` taxonomy.
  *
  * @return array Contains objects with information about a term (name, link)
  */
private function getTaxonomyName()
{
    $terms = $this->getTermData('taxonomy-slug');

    return $terms;
}
```

In `getData` you can then call it as follows.

```php
// Using a named helper
$data->taxonomyName = $this->getTaxonomyName();

// Directly using getTermData
$data->otherTaxonomy = $this->getTermData('other-taxonomy');
```

## Usage

Make sure to format the `$data` variable you return in a way that your templating system understands. Then wherever you want to make use of this data do the following.

```php
// examplary: this assumes you are in a loop
$postId = get_the_ID();
$postDataObject = new Helpers\PostData($postId);
$postData = $postDataObject->getData();

// Now hand it to your templates
```
