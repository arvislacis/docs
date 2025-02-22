---
title: "Architecture Concepts: Composer"
description: "Documentation on using Composer within Winter CMS projects."
---

# Using Composer

## Introduction

Using [Composer](https://getcomposer.org/) as an alternative package manager to using the standard one-click update manager is recommended for more advanced users and developers.

Composer is the de-facto standard for package management in the PHP ecosystem, and can handle the downloading, installation and management of Winter CMS plugins and themes, as well as third-party Laravel packages and vendor libraries.

## Installing Winter via Composer

Installing Winter via Composer is easy. You can use the `create-project` command through Composer to quickly set up a new Winter installation.

```bash
composer create-project wintercms/winter <your installation directory> [version]

# Example:
#   composer create-project wintercms/winter mywinter
# or
#   composer create-project wintercms/winter ./ "dev-develop"
```

### Configuring Winter

If you have used `create-project` to create your Winter project it will automatically run the following commands for you:

- [`winter:install`](../console/setup-maintenance#install-winter-via-command-line) (the CLI installation wizard)
- [`winter:env`](../console/setup-maintenance#configure-winter-through-an-environment-file) (populates the `.env` file from the configuration files)
- [`winter:mirror public --relative`](../console/setup-maintenance#mirror-public-files) (sets up the project to use a public folder, recommended for security)

You can either go through this wizard to configure your project or you can cancel with (`Ctrl+C`) and manually reviewing and make changes to the configuration files located in `config/*.php`. If you take the manual approach, note that you will also need to run `php artisan migrate` to migrate the database yourself after configuring the project.

> **NOTE:** When running commands on your Winter project, make sure that you are located in the project root directory first (following the previous example you can run `cd mywinter`).

### Completing installation

Once the above commands have been run, refer to the [Post Installation steps](../setup/installation#post-installation-steps) on the Installation page to complete the process.

## Installing a plugin or theme using Composer

Using Composer to install plugins and themes in Winter CMS allows a degree of control over the versions of plugins in use, making it easy to synchronise and deploy Winter CMS to multiple environments.

You may install a theme or plugin through Composer by [publishing the plugin or theme](#publishing-plugins-or-themes) in [Packagist](https://packagist.org), the repository for Composer packages. Once the plugin or theme is published, you can include it in any Winter CMS project that has Composer enabled by running the following command within the root directory of the project:

```bash
composer require <your package name>

# Example:
#   composer require winter/wn-pages-plugin
```

If you wish to specify a particular version of the plugin or theme you want to include, you can specify the version as well:

```bash
composer require <your package name> "<version constraint>"

# Example:
#   composer require winter/wn-pages-plugin "^2.0.0"
```

You can also optionally specify a plugin or theme to be only included in "development" environments:

```bash
composer require --dev <your package name> "<version constraint>"

# Example:
#   composer require --dev winter/wn-builder-plugin "^2.0.0"
```

After installing any packages via composer it is important to ensure that any included migrations are run and if you are [using a public folder](../setup/configuration#using-a-public-folder) that the public folder is regenerated. Use the following commands to accomplish that:

```bash
php artisan migrate && php artisan winter:mirror public --relative
```

### Missing `composer.json`?

If you have created your Winter CMS project recently, using either Composer or the Web Installer, then you should have a `composer.json` present in your proejct root.

If you are missing your `composer.json` file, simply copy the latest [`composer.json`](https://github.com/wintercms/winter/tree/develop/composer.json) file from [GitHub](https://github.com/wintercms/winter/tree/develop) into your Winter instance and then run `composer install` within the root directory of the project.

> **NOTE:** If you have made modifications to the files within the `modules` directory, these will be overwritten by Composer if an update to those modules is installed. It is recommended that you *do not* make modifications to the modules directly.

### Development branch

If you plan on submitting pull requests to the Winter CMS project via GitHub, or are actively developing a project based on Winter CMS and want to stay up to date with the absolute latest version, we recommend switching your composer dependencies to point to the `develop` branch where all the latest improvements and bug fixes take place. Doing this will allow you to catch any potential issues that may be introduced (as rare as they are) right when they happen and get them fixed while you're still actively working on your project instead of only discovering them several months down the road if they eventually make it into production.

```json
"winter/storm": "dev-develop as 1.2.999",
"winter/wn-system-module": "dev-develop as 1.2.999",
"winter/wn-backend-module": "dev-develop as 1.2.999",
"winter/wn-cms-module": "dev-develop as 1.2.999",
"laravel/framework": "~9.0",
```

### Deployment best practices

Using the following best practices with Composer and Winter CMS will make deployment of your Winter CMS installation much smoother:

- Store the `composer.lock` file in your source control. This file is ignored by Git by default in the `.gitignore` file included by Winter CMS, but should be deployed with your application to speed up the install and update process.
- Add a `.gitignore` file inside the `modules` folder to ignore all changes within this folder, as the modules will be installed and updated by Composer.
- Add a `.gitignore` file inside the `plugins` folder to ignore all changes within this folder if you install your plugins via Composer. You can optionally allow custom plugins that are only being used for that specific project.
- Use `composer install --no-dev` on your production instance to specifically exclude any "development" packages and libraries that won't be used in production.

## Publishing plugins or themes

When publishing your plugins or themes to the marketplace, you may wish to also make them available via Composer. An example `composer.json` file for a plugin is included below:

```json
{
    "name": "winter/wn-demo-plugin",
    "type": "winter-plugin",
    "description": "Demo Winter CMS plugin",
    "keywords": ["winter", "cms", "demo", "plugin"],
    "license": "MIT",
    "authors": [
        {
            "name": "Winter CMS Maintainers",
            "url": "https://wintercms.com",
            "role": "Maintainer"
        }
    ],
    "require": {
        "php": ">=7.2",
        "composer/installers": "~1.0"
    }
}
```

Be sure to start your package `name` with **wn-** and end it with **-plugin** or **-theme** respectively - this will help others find your package and is in accordance with the [quality guidelines](./developer-guide#repository-naming).

The `type` field is a key definition for ensuring that your plugin or theme arrives at the correct location upon installation. Use the following types:

Product | Type
------- | -------------
Plugin  | `winter-plugin`
Theme   | `winter-theme`
Module  | `winter-module`

> **Reminder**: Be sure to specify any dependencies in your `composer.json` file as you would using the  `$require` property found in the [plugin registration file](../plugin/registration#dependency-definitions)

## Package descriptions

There are many different moving parts that come together to make the Winter CMS platform work. Here we will describe the various packages you will likely encounter:

- **Modules** are the core packages that are included with Winter, you can think of them as "internal plugins" that provide core functionality. Modules use the package type `winter-module` and are located within the `/modules` directory. They are loaded manually via configuration and at least one module must be present in the `cms.loadModules` configuration item for the system to operate.

- **Plugins** extend the core functionality of Winter and are packages of type `winter-plugin`. They are located within the `/plugins` directory. The `System` module is responsible for the loading of plugins which happens automatically when found in the file system, unless they are explicitly disabled.

- **Themes** contain static file content that is used to manage the front end structure of your website and use the package type `winter-theme`. They are located within the `/themes` directory. The `Cms` module is responsible for managing themes and determining what theme is currently active.

- **Vendor** packages are included via Composer in either the project's `/vendor` directory or can sometimes be found in plugin-specific `/vendor` directories. The project vendor directory takes priority over and plugin vendor directories that appear in the system.

> **NOTE:** Refer to the [Architecture documentation](../architecture/introduction) for more detailed information on how Winter CMS is structured.

## Marketplace builds

When you publish your plugin or theme to the marketplace, the server will conveniently pull in all the packages defined in your composer file. This makes the product ready for others to use, even if they don't use composer. Here's how it works:

1. As a plugin or theme developer, you can define your external dependencies and packages in `composer.json`, including other plugins or themes.

2. The server will attempt to remove any core dependencies that are inherently available in the core, including Laravel, Winter and its related packages.

3. The server will run  `composer install` in your plugin or themes directory, pulling dependencies into the `vendor` directory, local to that package.

4. The files `composer.json` and `composer.lock` are then removed to prevent the package files from becoming duplicated and a potential double up of dependencies.

5. The final result is packaged up and ready for consumption by Winter CMS platforms using one-click updates.

It is a good idea not to include the `vendor` directory when publishing your plugin or theme to the marketplace, the server will handle this for you.

If you are developing with your plugin, you can run `composer update` from the root directory. A special package called `wikimedia/composer-merge-plugin` will scan the plugins directory and merge the dependencies in to the main composer file.

## Using Laravel packages

When including Laravel packages in Winter CMS plugins there are a few things to take note of.

### Configuration files

Laravel packages will often provide configuration files, and they will usually come with the instructions to publish these config files to the project config folder, usually something like `php artisan vendor:publish --tag=config`.

However, this can create problems with Winter's plugin oriented design, since there would now be random config files in the core `/config` directory. In order to solve this problem, it is recommended that you proxy the included package's configuration through your plugin instead.

You may place this code in your Plugin registration file and call it from the  the `boot()` method.

```php
public function bootPackages()
{
    // Get the namespace code of the current plugin
    $pluginNamespace = str_replace('\\', '.', strtolower(__NAMESPACE__));

    // Locate the packages to boot
    $packages = \Config::get($pluginNamespace . '::packages');

    // Boot each package
    foreach ($packages as $name => $options) {
        // Apply the configuration for the package
        if (
            !empty($options['config']) &&
            !empty($options['config_namespace'])
        ) {
            Config::set($options['config_namespace'], $options['config']);
        }
    }
}
```

Now you are free to provide the packages configuration values the same way you would with regular plugin configuration values.

```php
return [
    // Laravel Package Configuration
    'packages' => [
        'packagevendor/packagename' => [
            // The accessor for the config item, for example,
            // to access via Config::get('purifier.' . $key)
            'config_namespace' => 'purifier',

            // The configuration file for the package itself.
            // Copy this from the package configuration.
            'config' => [
                'encoding'      => 'UTF-8',
                'finalize'      => true,
                'cachePath'     => storage_path('app/purifier'),
                'cacheFileMode' => 0755,
            ],
        ],
    ],
];
```

Now the package configuration has been included natively in Winter CMS and the values can be changed normally using the [standard configuration approach](../plugin/settings#file-based-configuration).

### Aliases & service providers

By default, Winter CMS disables the loading of discovered packages through [Laravel's package discovery service](https://laravel.com/docs/9.x/packages#package-discovery), in order to allow packages used by plugins to be disabled if the plugin itself is disabled. Please note that packages defined in `app.providers` will still be loaded even if discovery is disabled.

> **NOTE:** It is possible to set `app.loadDiscoveredPackages` to `true` in the project configuration to enable automatic loading of these packages. This will result in packages being loaded, even if the plugin using them is disabled. This is **NOT RECOMMENDED.**

In order to manually register ServiceProviders and Aliases provided by external Laravel packages that are used by your plugins you should use the `App` facade and `AliasLoader` instance respectively:

```php
use App;
use Illuminate\Foundation\AliasLoader;
use System\Classes\Plugin as PluginBase;

class Plugin extends PluginBase
{
    public function register()
    {
        // Instantiate the AliasLoader
        $aliasLoader = AliasLoader::getInstance();

        // Register the aliases provided by the packages used by your plugin
        $aliasLoader->alias('Purifier', \Mews\Purifier\Facades\Purifier::class);

        // Register the service providers provided by the packages used by your plugin
        App::register(\Mews\Purifier\PurifierServiceProvider::class);
    }
}
```

### Migrations & models

Laravel packages that interact with the database will often include their own database migrations and Eloquent models. Ideally you should duplicate these migrations and models to your plugin's directory and then rebase the provided Model classes to extend the base `\Winter\Storm\Database\Model` class instead of the base Laravel Eloquent model class to take advantage of the extended technology features found in Winter.

You should also make an effort to rename the tables to prefix them with your plugin's author code and name. For example, a table with the name `posts` should be renamed to `winter_blog_posts`.

## Merging plugin Composer dependencies

By default, Winter CMS includes the [Composer Merge plugin](https://github.com/wikimedia/composer-merge-plugin) with its Composer dependencies. This is a special Composer plugin that allows developers to include a `composer.json` file in plugins that they are actively developing, or do not wish to publish, and have any dependencies specified in their plugin's `composer.json` file also be included when running `composer install` or `composer update` on their project.

Originally, this was set to include any plugin's `composer.json` file, but this was prone to conflicts and errors, so beginning with Winter 1.2, you must explicitly provide a list of `composer.json` files that you wish to merge in with your main `composer.json` file.

You can edit the `include` section of the following block of code in your **main** `composer.json` file:

```json
"extra": {
    "merge-plugin": {
        "include": [
            "plugins/myauthor/*/composer.json"
        ],
        "recurse": true,
        "replace": false,
        "merge-dev": false
    }
},
```

For example, if you want to include a single plugin's `composer.json`, you can specify a direct path to that plugin's `composer.json`:

```json
"extra": {
    "merge-plugin": {
        "include": [
            "plugins/acme/blog/composer.json"
        ],
        "recurse": true,
        "replace": false,
        "merge-dev": false
    }
},
```

Or if you have multiple plugins, you can specify multiple paths:

```json
"extra": {
    "merge-plugin": {
        "include": [
            "plugins/acme/blog/composer.json",
            "plugins/acme/forum/composer.json",
            "plugins/acme/events/composer.json"
        ],
        "recurse": true,
        "replace": false,
        "merge-dev": false
    }
},
```

If you wish to include a whole directory of custom plugins, you can specify a wildcard:

```json
"extra": {
    "merge-plugin": {
        "include": [
            "plugins/acme/*/composer.json"
        ],
        "recurse": true,
        "replace": false,
        "merge-dev": false
    }
},
```

To prevent conflicts, you must not include any plugins that are brought in as a dependency of the project in either the `require` or `require-dev` blocks of your project's `composer.json` file. For example, if you have previously used `composer require acme/blog` to add that plugin to your `require` block, you should not include it in the Merge plugin's `include` block, as this effectively includes the plugin twice and may cause Composer to see it as a conflict.
