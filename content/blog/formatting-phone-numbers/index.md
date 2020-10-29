---
title: Formatting phone numbers
date: 2020-10-29T20:30
---

This article shows how to format a string of numbers and format it for use in `href` attributes and make them more readable for visitors. The concrete examples show how to integrate this with WordPress however the principles can be applied to any application.

## Clean up the string

```php
<?php
function cleanPhoneNumber($phone_number) {
    // Strip out all spaces
    $number = trim(str_replace(' ', '', $phone_number));

    // Make sure there is no (0) in the phone number
    $number = str_replace('(0)', '', $number);

    // This is an educated guess about whether or not we have an international prefix.
    if (substr($number, 0, 1) !== '+' && substr($number, 0, 1) !== '00') {
        // If the number starts with a 0 this is probably the region prefix, which must be removed
        // if there is an international prefix.
        if (substr($number, 0, 1) === '0') {
            $number = substr($number, 1);
        }

        $number = '+41' . $number;
    }

    return $number;
}
```

This function, which in the WordPress environment can be hooked up to a custom filter, cleans up a given number according to a few rules.

1. Remove whitespace and trim the number.
2. Some people enter the state prefix as `(0)`. Since we add an international prefix where this is unecessary anyways, we just strip it.
3. We are checking for an international prefix. The function assumes it exists if the string starts with `+` or `00` which is the common way to write these in Switzerland. Of course to be cleaner you could check against a database, in any case make sure to adjust this to your local customs.
4. If there is no prefix, check the first number, if it is a 0 we assume it to be the state prefix which needs to be removed.
5. If there is no international prefix we add the Swiss prefix by default. If necessary add this as a function argument or check the region programmatically and set it from that information.

We expect the function to turn i. e. `012 345 67 89` (custom Swiss phone number format without international prefix) into `+41123456789`. The output is the same if the number is input as `0041 12 345 67 89` or `+41 12 345 67 89`.

## Format the string

To get consistent and readable output of phone numbers you can make use of this second function.

```php
function formatPhoneNumber($phone_number) {
    // Add a fallback, in case these transformations do not return a result
    // (i. e. the phone number format is not recognized).
    $result = $phone_number;

    /**
     * Clean up the phone number (make sure we have no spaces, an international prefix, etc.).
     *
     * Call it directly or use the filter
     */
    $number = cleanPhoneNumber($phone_number);

    /**
     * Do some checks for possible phone number formats using preg_match, then add spaces where necessary.
     */

    // Swiss phone number format.
    if (preg_match('/^(\+\d{2})(\d{2})(\d{3})(\d{2})(\d{2})$/', $number, $matches)) {
        $result = $matches[1] . '&nbsp;' .
            $matches[2] . '&nbsp;' .
            $matches[3] . '&nbsp;' .
            $matches[4] . '&nbsp;' .
            $matches[5]
        ;
    }

    // German phone number format.
    if (preg_match('/^(\+\d{2})(\d{4})(\d{6})(\d{1,2})$/', $number, $matches)) {
        $result = $matches[1] . '&nbsp;' .
            $matches[2] . '&nbsp;' .
            $matches[3] . '&nbsp;' .
            $matches[4];
    }

    return $result;
}
```

Again in a WordPress environment you could add this function to custom filter. Here is what the function is set up to do in the example.

1. Store the original input in a variable in case we cannot return valid output.
2. Clean up the given string to make sure it has an international prefix and no spaces. Call the above function (`cleanPhoneNumber`) directly or use the filter you hooked it to if in WordPress.
3. Do some `preg_match` checks, currently for Swiss and German phone number format. What we do is check for number groupings which are then stitched together with non breaking spaces. The expected result is `+41&nbsp;12&npbs;345&nbsp;67&nbsp;89`. For German numbers the groupings and length differs, but it's the same process.

You can add more `preg_match` checks for other formats. Manually check the international prefix if phone numbers have the same length but expect different groupings.

## Example usage

This is how you would use these helper functions in a WordPress environment. Say we want to display team members and one piece of information we offer is their phone number stored using an Advanced Custom Field. We put the two functions as public static methods into a Helper class. When collecting data to output you can do the following.

```php
<?php
$theNumber = get_field('phone_number', $postId);
$cleanNumber = Helper::cleanPhoneNumber($theNumber);
$formatNumber = Helper::formatPhoneNumber($theNumber);
$phoneLink = [
  'href' => esc_url('tel:' . $cleanNumber),
  'text' => esc_html(decode_html_entities($formatNumber)),
];
```

Then use this in your views as your templating language expects, or using just PHP:

```php
<?php
echo '<a href="' . $phoneLink['href'] . '>' . $phoneLink['text'] . '</a>';
```
