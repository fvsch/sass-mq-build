sass-mq-build
=============


A simple [Sass](http://sass-lang.com/) function to build media query strings that can be stored in variables, to be used with the native CSS `@media (…)` syntax.

This approach is inspired by the [CSS Custom Media Queries proposal](https://www.w3.org/TR/2016/WD-mediaqueries-4-20160126/#custom-mq) (which may or may not end up in Media Queries Level 5. It allows using this kind of syntax in Sass code:

```scss
$my-media-query: mq-build(30em, 60em);

@media ($my-media-query) {
  /* Nice CSS code */
}
```

which outputs:

```css
@media (min-width: 30em) and (max-width: 59.99em) {
  /* Nice CSS code */
}
```


Features
--------

-   You can use `@media` instead of `@include` with a mixin, yay!
-   Accepts any CSS length unit (`px`, `em` and more) for breakpoints.
-   Can substract `1px` or `0.01em` from “max” values (`max-width` and `max-height`). This is helpful if you want to avoid situations were two conflicting media queries match because the window width (or height) is *exactly* at the provided dimension.
-   Supports `min-width`, `max-width`, `min-height`, `max-height`, and passing other arbitrary queries as strings (e.g. `"orientation: landscape"`). This is limited but works for 99.9% of our needs (and actually we mostly use min-width/max-width and not much else).

For alternatives and/or bigger feature set, you might want to take a look at [@include-media](http://include-media.com/) or [Sass MQ](https://github.com/sass-mq/sass-mq).


Complete usage
--------------

The `mq-build()` function can particularly be useful if you’re using variables to store breakpoints, and want to create a set of variables for pre-built media queries.

```scss
@import "mq-build";

// Two breakpoints divide the possible widths in 3 groups
$start-medium: 750px;
$start-large: 1280px;

// Single-group queries
$mq-small:  mq-build($maxw: $start-medium);
$mq-medium: mq-build($minw: $start-medium, $maxw: $start-large);
$mq-large:  mq-build($minw: $start-large);

// Two-group queries
$mq-upto-medium: mq-build($maxw: $start-large);
$mq-from-medium: mq-build($minw: $start-medium;
```

Then in your CSS code…

```scss
.MyComponent {
  /* Common, mobile-first styles */
  @media ($mq-from-medium) {
    /* Styles for medium and large screens */
  }
  @media ($mq-large) {
    /* Large screens only */
  }
}
```

Note that you can always combine a custom media query (saved in a variable) with a literal one:

```scss
@media ($desktop) and (min-resolution: 2ddpx) {
  /* High resolution big screen */
}
```


Function parameters
-------------------

All parameters are optional.

1.  `$minw` - CSS length value for `min-width`
2.  `$maxw` - CSS length value for `max-width`
3.  `$minh` - CSS length value for `min-height`
4.  `$maxh` - CSS length value for `max-height`
5.  `$other` - (string or list of strings)
6.  `$wrap` - whether to wrap the result in parentheses

If the resulting query has more than one condition, the `and` keyword is used. There is no support for `or`, `not`, etc.


Global options
--------------

### `$mq-build-wrap` (defaults to `false`)

Should we add parentheses around the returned string? This is theoretically better, but it doesn’t work with the `@media ($variable)` syntax.

If you set this option to true, or pass `mq-build(…, $wrap:true)`, you will have to use the generated string like this:

```scss
@media #{$variable} { … }
```

### `$mq-build-offset` (defaults to `true`)

Whether to remove a small number from `max-width` and `max-height` queries, in order to avoid collisions. What collisions? Well, if you have this kind of code:

```css
@media (min-width: 800px) {
    ...
}
@media (max-width: 800px) {
    ...
}
```

… then your intention was probably *not* to have both blocks of CSS apply at the same time. But if the window width is exactly 800px, both media queries *will* match, resulting in some style conflicts.

So our solution is to automatically output this code:

```css
@media (min-width: 800px) {
    ...
}
@media (max-width: 799px) {
    ...
}
```
