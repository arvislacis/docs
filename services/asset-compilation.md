---
title: "Services: Asset Compilation"
description: "Learn more about the Asset Compilation features included with Winter CMS."
---
# Asset Compilation

## Introduction

Winter CMS includes a server-side Asset Compiler that makes use of the [Assetic Framework](https://github.com/assetic-php/assetic) to compile and combine assets like CSS and JavaScript serverside, through PHP, negating the need for complex build workflows. The Asset Compiler provides on-the-fly server-side compilation of SASS and LESS stylesheets as well as [run-once manual compilation of assets](#compiler-bundles) without requiring additional workflow tools like Node or NPM. It is also able to combine and minify CSS and JS files.

Additionally, you can [define variables in the theme.yaml file](../themes/development#asset-compiler-variables) that can be modified in the Theme Settings area of the backend which are then injected into the compiled files, creating flexibility for theming and branding.

> **NOTE:** Looking for a way to compile or build Node-based assets in your project? Check out the [Winter Mix / Vite / NPM](../console/asset-compilation) functionality which can handle this for you.

## Injecting Page Assets

Asset injection into the generated HTML of Winter CMS powered sites is handled by the `System\Traits\AssetMaker` trait. This trait is used in several places in Winter CMS to interact with the Asset Compiler from directly within a class (usually one that interacts with the generated HTML) and is the primary method of interacting with the Asset Compiler. Classes that currently implement the trait include the following:

- `ControllerBehavior`
- `Widget`
- `Component`
- Backend Controllers
- Frontend Controller (through the `| theme` filter when provided an array)

This trait provides the following methods:

- `addJs($name, $attributes = [])`
- `addCss($name, $attributes = [])`

```php
// Add a publicly accessible asset to the injected page assets
$this->addJs('/plugins/acme/blog/assets/javascript/blog-controls.js');
```

If the path specified in the `addCss` and `addJs` method argument begins with a slash (/) then it will be relative to the website root. If the asset path does not begin with a slash then it is relative to the directory referenced by the `$assetPath` property on the class currently implementing the `System\Traits\AssetMaker`.

### System Attribute: `build`

The `addCss` and `addJs` methods provide a second argument that defines the attributes of your injected asset as an array. A special attribute - `build` - is available, that will suffix your injected assets with the current version of the plugin specified. This can be used to refresh cached assets when a plugin is upgraded.

```php
$this->addJs('/plugins/acme/blog/assets/javascript/blog-controls.js', [
    'build' => 'Acme.Test',
    'defer' => true
]);
```

You may also use a string as the second argument, which then defaults to using the string value as the value for the `build` option:

```php
$this->addJs('/plugins/acme/blog/assets/javascript/blog-controls.js', 'Acme.Test');
```

### System Attribute: `order`

Another special attribute `order` is reserved to be used when rendering the assets to ensure that assets can be rendered in a specific order. The default order priority assigned to most assets is `500` and assets with the same priority are ordered separately on a first-come first-served basis.

>**NOTE:** Core assets provided by the current backend skin are generally provided with an order priority of `1` as they are technically added to the AssetMaker after the rest of the page has been processed, immediately before the call to `makeAssets()` but they need to be loaded before any assets added to the page dynamically. This is so that you can still use the `system.assets.beforeAddAsset` global event and / or `assets.beforeAddAsset` local event to interact with assets provided by the backend skin.

### Triggering Asset Compilation

In order to trigger asset compilation or combination, asset paths must be passed as an array of paths to the above methods, even if there is just a single asset file to compile that itself loads other files. Asset Compilation is currently supported for JS files using the `=require path/to/other.js` syntax as well as LESS and SASS/SCSS files.

Example:

```php
$this->addCss(['assets/less/base.less']);
```

### Combining CSS or JavaScript Files

The Asset Compiler can also be used to combine assets of the same type by passing an array of files.

```php
$this->addCss([
    'assets/css/styles1.css',
    'assets/css/styles2.css'
]);
```

> **NOTE**: You can enable the assets minification with the `cms.enableAssetMinify` configuration value in the `config/cms.php` file. By default minification is disabled.

### Path Symbols

In some cases you may wish to combine a file outside of the current context (AssetMaker trait container or the current theme), this can be achieved by prefixing the path with a symbol to create a dynamic path. For example, a path beginning with `~/` will create a path relative to the application:

```html
<script src="{{ ['~/modules/system/assets/js/framework.js'] | theme }}"></script>
```

The following symbols are supported for creating dynamic paths:

Symbol | Description
------------- | -------------
`~` | Relative to the application directory
`$` | Relative to the plugins directory
`#` | Relative to the themes directory

### Combiner Aliases

The asset combiner supports common aliases that substitute file paths, these will begin with the `@` symbol. For example the [AJAX framework assets](../ajax/introduction#including-the-framework) can be included in the combiner:

```twig
<script src="{{ [
    '@jquery',
    '@framework',
    '@framework.extras',
    'assets/javascript/app.js'
] | theme }}"></script>
```

The following aliases are supported:

Alias | Description
------------- | -------------
`@jquery` | Reference to the jQuery library (v3.7.1) used in the backend. (JavaScript)
`@framework` | AJAX framework extras, subsitute for `{% framework %}` tag. (JavaScript)
`@framework.extras` | AJAX framework extras, subsitute for `{% framework extras %}` tag. (JavaScript, CSS)
`@framework.extras.js` | AJAX framework extras, (JavaScript)
`@framework.extras.css` | AJAX framework extras, (CSS)

The same alias can be used for JavaScript or CSS, for example `@framework.extras`. At least one explicit reference with a file extension is needed in the array to determine which is used.

### Rendering Injected Assets

In order to output the injected assets on the frontend you need to use the [{% styles %}](/docs/v1.2/markup/tags/styles) and [{% scripts %}](/docs/v1.2/markup/tags/scripts) tags.

Example:

```twig
<head>
    ...
    {% styles %}
</head>
<body>
    ...
    {% scripts %}
</body>
```

If you are wanting to render the injected assets in any other context, you can call `$this->makeAssets()` on the object that implements the `System\Traits\AssetMaker` trait to rener the injected assets for that object instance.

> **NOTE:** Injected assets are rendered by default in the backend through the `<?= $this->makeAssets() ?>` call in `modules/backend/layouts/_head.htm`, so if you are using a custom layout for your backend controllers you will need to ensure that it includes that call.

## Compiler Bundles

While the majority of the time dynamic asset compilation through `addJs()`, `addCss()`, or the [`| theme` filter](/docs/v1.2/markup/filters/theme) should be sufficient for your needs, you may occassionally have a complex asset compilation that you would like to just generate a static file on command instead of dynamically.

The Winter CMS core registers several such bundles for internal usage that are compiled whenever the [`artisan winter:util compile assets` command](../console/utilities#compile-winter-assets) is run.

## Extending the Asset Compiler

The `System\Classes\AssetCombiner` class provides the `registerCallback(callable $callback)` static method to extend it's default behaviour. Additionally, the [`system.assets.beforeAddAsset` event](/docs/v1.2/api/events/system/assets/beforeAddAsset) is available for extending any calls to `addJs()` or `addCss()`

### Register Custom Aliases

It is possible to register your own custom aliases through the `registerCallback` method of the `System\Classes\CombineAssets` class:

```php
/*
 * Register custom asset compiler aliases
 */
CombineAssets::registerCallback(function ($combiner) {
    $this->registerAlias('jquery', '~/modules/backend/assets/js/vendor/jquery-and-migrate.min.js');
});
```

### Register Custom Asset Bundles

It is possible to register your own custom asset bundles through the `registerCallback` method of the `System\Classes\CombineAssets`:

```php
/*
 * Register custom asset compiler bundles
 */
CombineAssets::registerCallback(function ($combiner) {
    $combiner->registerBundle('~/modules/system/assets/less/styles.less', '$/myauthor/myplugin/assets/css/generate-styles.css');
});
```
