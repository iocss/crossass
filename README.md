# Crossass

A tiny Sass mixin / function library for modular CSS.

## Features

* Module definition
* Class exporting
* Modifier support
* Multiple inheritance
* Selectable extending
* Selector Helper
* [BEM](http://bem.info/) syntax support

## Requirement

* [Sass](http://sass-lang.com/) 3.3.0+

##Usage

### Import

```scss
@import "crossass/scss/core";
```

### Module (Placeholder definition)

`x-module` mixin produces ruleset(s) that use [Placeholder Selector](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholder_selectors_).

SCSS:
```scss
@include x-module('module') {
  & {
    border: 1px solid #000;
    strong {
      color: #f00;
    }
  }
}
.module {
  @extend %module;
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module strong {
    color: #f00;
}
```

### Module with class exporting

By passing `true` to 2nd parameter, you can also export ruleset(s) that use class selector so you don't have to use `@extend` to export them.

SCSS:
```scss
@include x-module('module', true) {
  & {
    border: 1px solid #000;
    strong {
      color: #f00;
    }
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module strong {
    color: #f00;
}
```

### Modifier

`x-modifier` mixin generates a suffixed selector.
The default modifier pattern is `$x-modifier-separator` + the modifier name and the default `$x-modifier-separator` is `-` (see also [Variables](#variables)).

SCSS:
```scss
@include x-module('module', true) {
  & {
    border: 1px solid #000;
  }
  @include x-modifier('important') {
    border-color: #f00;
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module-important {
    color: #f00;
}
```

### Extending

`x-extend` mixin is similar with `@extend`.

SCSS:
```scss
@include x-module('module') {
  & {
    border: 1px solid #f00;
  }
}
.module {
  @include x-extend('module');
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
```

### Extending (multiple inheritance)

`x-extend` mixin allows you to extend a ruleset from multiple rulesets.

SCSS:
```scss
@include x-module('module-1') {
  & {
    border: 1px solid #f00;
  }
}
@include x-module('module-2') {
  & {
    background: #000;
  }
}
.module {
  @include x-extend('module-1', 'module-2');
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module {
    background: #000;
}
```

You cannot specify the order of multiple inheritance and **Sass outputs rulesets in order of the definition**.
This is a restriction of Sass so **the order of ruleset definition is very important** with Crossass.

SCSS:
```scss
@include x-module('module-1') {
  & {
    border: 1px solid #f00;
    background: #f00;
  }
}
@include x-module('module-2') {
  & {
    background: #000;
  }
}
.module {
  @include x-extend('module-1', 'module-2');
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
    background: #f00;
}
.module {
    background: #000;
}
```

### Extending with specific module element(s)

You can extend a ruleset with specific module element at once.

SCSS:
```scss
@include x-module('block') {
  & {
    border: 1px solid #000;
  }
  @include x-modifier('header') {
    font-size: 1.5em;
  }
  @include x-modifier('body') {
    margin: 1em;
  }
  @include x-modifier('footer') {
    background: #ccc;
  }
}
@include x-module('alert', true) {
  @include x-extend((
    block: ('-header', '-body')
  ));
  & {
    background: #f00;
  }
}
```

CSS:
```css
.alert {
    border: 1px solid #000;
}
.alert-header {
    font-size: 1.5em;
}
.alert-body {
    margin: 1em;
}
.alert {
    background: #f00;
}
```

You can still use multiple inheritance.

SCSS:
```scss
@include x-module('block-1') {
  & {
    border: 1px solid #000;
  }
  @include x-modifier('header') {
    font-size: 1.5em;
  }
  @include x-modifier('body') {
    margin: 1em;
  }
  @include x-modifier('footer') {
    background: #ccc;
  }
}
@include x-module('block-2') {
    @include x-modifier('footer') {
        font-size: .75em;
    }
}
@include x-module('alert', true) {
  @include x-extend((
    block-1: ('-header', '-body'),
    block-2: ('-footer'),
  ));
  & {
    background: #f00;
  }
}
```

CSS:
```css
.alert {
    border: 1px solid #000;
}
.alert-header {
    font-size: 1.5em;
}
.alert-body {
    margin: 1em;
}
.alert-footer {
    font-size: .75em;
}
.alert {
    background: #f00;
}
```

### Selector Helper

`x-modifier` function is useful if you want to generate the selector for a module element.
This function returns the selector that is `$x-modifier-separator` + a modifier name (see also [Variables](#variables)).

SCSS:
```scss
@include x-module('block') {
  & {
    border: 1px solid #000;
  }
  @include x-modifier('body') {
    margin: 1em;
  }
}
@include x-module('alert', true) {
  @include x-extend((
    block: (x-modifier('body'))
  ));
  & {
    background: #f00;
  }
}
```

CSS:
```css
.alert {
    border: 1px solid #000;
}
.alert-body {
    margin: 1em;
}
.alert {
    background: #f00;
}
```

`x-modifier` function can generate multiple selectors at once.
In Addition, you can also use multiple `x-modifier` function in `x-extend` mixin.

SCSS:
```scss
@include x-module('block') {
  & {
    border: 1px solid #000;
  }
  @include x-modifier('header') {
    font-size: 1.5em;
  }
  @include x-modifier('body') {
    margin: 1em;
  }
  @include x-modifier('footer') {
    background: #ccc;
  }
}
@include x-module('alert', true) {
  @include x-extend((
    block: (
      x-modifier('header', 'body'),
      x-modifier('footer')
  ));
  & {
    background: #f00;
  }
}
```

CSS:
```css
.alert {
    border: 1px solid #000;
}
.alert-header {
    font-size: 1.5em;
}
.alert-body {
    margin: 1em;
}
.alert-footer {
    background: #ccc;
}
.alert {
    background: #f00;
}
```

### Variables

Variable | Default | Description
--- | --- | ---
$x-module-exporting | false | Whether `x-module` mixin should export ruleset(s) that use class selector
$x-modifier-separator | '-' | The separator for modifiers
$x-strict-extend | false | `x-extend` mixin never uses `!optional` if true

## BEM syntax support

Crossass also supports [BEM](http://bem.info/) syntax with BEM extension.

### Import

```scss
@import "crossass/scss/core";
@import "crossass/scss/extension/bem";
```

### Mixins

* `x-bem-modifier`
* `x-bem-element`

The usage is the same as `x-modifier` mixin.

### Selector Helper

* `x-bem-modifier`
* `x-bem-element`

The usage is the same as `x-modifier` function.

### Variables

Variable | Default | Description
--- | --- | ---
$x-bem-modifier-separator | '--' | The separator for modifiers
$x-bem-element-separator | '__' | `The separator for elements
