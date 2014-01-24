# Crossass

A Sass mixin / function library for modular CSS.

![Crossass Inspector](http://raw.github.com/whizark/crossass/master/crossass-inspector.png)

## Features

* Module definition
* Class exporting
* Modifier support (also the nesting)
* Parent ruleset inheritance
* Multiple inheritance
* Selectable extending
* Selector Helper
* BEM syntax support
* Variable inspection (Crossass Inspector)
* Automatic `@extend` of parent module's children (in progress)
* No dependencies except for Sass

## Requirement

Crossass is a pure Sass mixin / function library, so really portable.

* [Sass](http://sass-lang.com/) 3.3.0+

## Background

To build moudlar CSS, you know there are some popular methodologies like [OOCSS](http://oocss.org/) / [SMACSS](http://smacss.com/) / [BEM](http://bem.info/) etc.
In [Sass](http://sass-lang.com/) 3.0.0+, we can build CSS using BEM syntax like the following.

SCSS::
```scss
.nav {    // Module
    @at-root {
        & {
            // ...
        }
        #{&}__menu {    // Module element
            // ...
        }
        #{&}--global {    // Module variation
            // ...
        }
    }
}
```

CSS:
```css
.nav {
    // ...
}
.nav__menu {
    // ...
}
.nav--global {
    // ...
}
```

It looks like nice but you will find out there are still a few issues.
For example, how can I add a child element into the module variation `.nav--global` ? Like this?

SCSS:
```scss
.nav {    // Module
    @at-root {
        & {
            // ...
        }
        #{&}__menu {    // Module element
            // ...
        }
        #{&}--global {    // Module variation
            // ...
            #{&}__menu {    // Module variation's element (?)
                // ...
            }
        }
    }
}
```

Actually, this works.

CSS:
```css
.nav {
    // ...
}
.nav__menu {
    // ...
}
.nav--global {
    // ...
}
.nav--global .nav--global__menu {
    // ...
}
```

However, it's bad for the eyes and it might also cause CSS specificity-related issue near the future for the `.nav--global .nav--global__menu`.

By using `@at-root` inside the child selector, this issue can also be solved.

SCSS:
```scss
.nav {    // Module
    @at-root {
        & {
            // ...
        }
        #{&}__menu {    // Module element
            // ...
        }
        #{&}--global {    // Module variation
            // ...
            @at-root {
                #{&}__menu {    // Module variation's element
                    // ...
                }
            }
        }
    }
}
```

CSS:
```css
.nav {
    // ...
}
.nav__menu {
    // ...
}
.nav--global {
    // ...
}
.nav--global__menu {
    // ...
}
```

Although you can do this, you have to repeatedly define the declarations for the elements
and the source code will get to be unreadable because of the nested module variation(s).

As another solution, you might come up with using `@extend`.

SCSS::
```scss
.nav {    // Module
    @at-root {
        & {
            // ...
        }
        #{&}__menu {    // Module element
            // ...
        }
    }
}
.nav--global {    // Module variation
    @extend .nav;
}
```

Unfortunately, this SCSS produces a sad result.

CSS:
```css
.nav, .nav--global {
    // ...
}
.nav__menu {
    // ...
}
```

Where is the module variation's element `.nav--global__menu` ...?

You should put the declarations for the element into `.nav__menu` together...?
Although it might not be a big issue while the module variation's element is the same as the parent module's element,
it will break up the principle of the modular CSS.
To extend parent module's elements, you hava to use `@extend` for the each elements.

Crossass aims to be a Sass mixin / function library for easily building modular CSS.

##Usage

### Import

```scss
@import "crossass/scss/environment/production";
@import "crossass/scss/core";
```

### Module (Placeholder definition)

`x-module` mixin produces ruleset(s) that use [Placeholder Selector](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholder_selectors_).

SCSS:
```scss
@include x-module('module') {
  border: 1px solid #000;
  strong {
    color: #f00;
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
  border: 1px solid #000;
  strong {
    color: #f00;
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
  border: 1px solid #000;
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
    border-color: #f00;
}
```

You can also define multiple selectors at once.

SCSS:
```scss
@include x-module('module', true) {
  border: 1px solid #000;
  @include x-modifier('information', 'important') {
    margin: 8px;
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module-information {
    margin: 8px;
}
.module-important {
    margin: 8px;
}
```

### Modifier (nesting)

`x-modifier` mixin also allows you to nest the declarations.

SCSS:
```scss
@include x-module('module', true) {
  border: 1px solid #000;
  @include x-modifier('alert') {
    border-color: #fa0;
    @include x-modifier('important') {
      border-color: #f00;
    }
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module-alert {
    border-color: #fa0;
}
.module-alert-important {
    border-color: #f00;
}
```

### Inheritance from parent ruleset

You can easily extend parent ruleset by using `x-parent` mixin.

SCSS:
```scss
@include x-module('alert', true) {
  border-width: 1px;
  border-style: solid;

  @include x-modifier('important') {
    @include x-parent();
    border-color: #fa0;
    font-weight: bold;

    @include x-modifier('danger') {
      @include x-parent();
      border-color: #f00;
    }
  }
}
```

CSS:
```css
.alert, .alert-important, .alert-important-danger {
  border-width: 1px;
  border-style: solid;
}
.alert-important, .alert-important-danger {
  border-color: #fa0;
  font-weight: bold;
}
.alert-important-danger {
  border-color: #f00;
}
```

### Extending

`x-extend` mixin is similar with `@extend`.

SCSS:
```scss
@include x-module('module') {
  border: 1px solid #f00;
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

### Extending (overriding)

You might override some declarations inside `x-extend` mixin.

SCSS:
```scss
@include x-module('module') {
  border: 1px solid #f00;
}
.module {
  @include x-extend('module') {
    border-width: 2px;
  }
}
```

CSS:
```css
.module {
    border: 1px solid #f00;
}
.module {
    border-width: 2px;
}
```

### Extending (multiple inheritance)

`x-extend` mixin allows you to extend a ruleset from multiple rulesets.

SCSS:
```scss
@include x-module('module-1') {
  border: 1px solid #f00;
}
@include x-module('module-2') {
  background: #000;
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
  border: 1px solid #f00;
  background: #f00;
}
@include x-module('module-2') {
  background: #000;
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
  border: 1px solid #000;
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
  background: #f00;
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
  border: 1px solid #000;
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
  background: #f00;
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
  border: 1px solid #000;
  @include x-modifier('body') {
    margin: 1em;
  }
}
@include x-module('alert', true) {
  @include x-extend((
    block: (x-modifier('body'))
  ));
  background: #f00;
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
In addition, you can also use multiple `x-modifier` function in `x-extend` mixin.

SCSS:
```scss
@include x-module('block') {
  border: 1px solid #000;
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
  background: #f00;
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
@import "crossass/scss/environment/production";
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
