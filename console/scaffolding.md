# Scaffolding Commands

The following commands allow you to quickly scaffold additional code into your Winter project, speeding up development time.

## Create a theme

```bash
php artisan create:theme <theme code> [scaffold]
```

The `create:theme` command generates a theme folder and basic files for the theme. The first argument specifies the theme code, eg. `myauthor-mytheme`. The second argument (`[scaffold]`) is optional and allows you to choose a base theme to start from:

- `tailwind`: (default) Creates an empty theme using [TailwindCSS](https://tailwindcss.com/) and [Vite](../console/asset-compilation-vite)
- `less`: Creates an empty theme using LESS and the [Asset Compiler](../services/asset-compilation)

## Create a plugin

```bash
php artisan create:plugin <plugin code>
```

The `create:plugin` command generates a plugin folder and basic files for the plugin. The first argument specifies the author and plugin name, eg. `MyAuthor.MyPlugin`.

## Create a component

```bash
php artisan create:component <plugin code> <component name>
```

The `create:component` command creates a new component class and the default component view. The first argument specifies the plugin code of the plugin that this component will be added into, and the second parameter specifies the component class name, eg. `MyComponent`.

## Create a migration

The `create:migration` command generates the migration file needed for a model database. The first argument specifies the plugin code of the plugin that this migration will be added into. Without any options, a bare migration file gets generated.

```bash
php artisan create:migration <plugin code>
```

In order to create a migration that auto-populates columns for your model, use the `--create` and `--model` options:

```bash
php artisan create:migration <plugin code> --create --model YourModel
```

In order to create an "update" migration, use the `--update` and `--model` options:

```bash
php artisan create:migration <plugin code> --update --model YourModel
```

## Create a model

```bash
php artisan create:model <plugin code> <model name>
```

The `create:model` command generates the files needed for a new model. The first argument specifies the plugin code of the plugin that this model will be added into, and the second parameter specifies the model class name, eg. `MyModel`.

## Create a factory

```bash
php artisan create:factory <plugin code> <factory name> --model=Post
```

The `create:factory` command generates a [Model Factory](https://laravel.com/docs/9.x/eloquent-factories#introduction) in the plugin's `database/factories` folder. The first argument specifies the plugin code of the plugin that this factory will be added into, and the second parameter specifies the name of the Factory class to generate. The `--m|model` option specifies the model that will be targeted by the factory, eg. `MyModel`.

## Create a settings model

```bash
php artisan create:settings <plugin code> [model name]
```

The `create:settings` command generates the files needed for a new [Settings model](../plugin/settings#database-settings). The first argument specifies the plugin code of the plugin that this model will be added into, and the second parameter is optional and specifies the Settings model class name (defaults to `Settings`).

## Create a backend controller

```bash
php artisan create:controller <plugin code> <controller name> [--sidebar]
```

The `create:controller` command generates a controller, configuration and view files. The first argument specifies the plugin code of the plugin that this controller will be added into, and the second parameter specifies the controller class name, eg. `MyController`.

The optional `--sidebar` flag will generate the controller with the Create, Update, & Preview views pre-configured to use the sidebar backend layout (like with the User Profile page).

## Create a form widget

```bash
php artisan create:formwidget <plugin code> <widget name>
```

The `create:formwidget` command generates a backend form widget, view and basic asset files. The first argument specifies the plugin code of the plugin that this form widget will be added into, and the second parameter specifies the form widget class name, eg. `MyFormWidget`.

## Create a report widget

```bash
php artisan create:reportwidget <plugin code> <widget name>
```

The `create:reportwidget` command generates a backend report widget, view and basic asset files. The first argument specifies the plugin code of the plugin that this report widget will be added into, and the second parameter specifies the report widget class name, eg. `MyReportWidget`.

## Create a job

The `create:job` command generates a job. The first argument specifies the plugin code of the plugin that this job will be added into, and the second parameter specifies the job class name, eg. `MyJob`.

```bash
php artisan create:job <plugin code> <job name>
```

By default the created job will be queueable and managed by queue worker.

The following options are supported:

short | long | description
----- | ---- | -----------
`-b` | `--batchable` | Generates a batchable queue job.
`-s` | `--sync` | Generates a non-queueable job.
`-f` | `--force` | Overwrites existing files with generated files
n/a | `--uninspiring` | Disables inspirational quotes

## Create a console command

```bash
php artisan create:command <plugin code> <command name>
```

The `create:command` command generates a [new console command](../console/introduction#building-a-command). The first argument specifies the plugin code of the plugin that this console command will be added into, and the second parameter specifies the command name.

## Create a test

```bash
php artisan create:test <plugin code> <path to class to be tested or test name>
```

The `create:test` command generates a test case. The first argument specifies the plugin code of the plugin that this job will be added into, and the second specifies the relative path to the class to be tested (i.e. a test for `\MyAuthor\MyPlugin\Classes\AuthManager` could be generated by calling `php artisan create:test myauthor.myplugin Classes\\AuthManager`) or the test's name (eg. `AuthManager`, which would be automatically expanded to `AuthManagerTest`).

The following options are supported:

short | long | description
----- | ---- | -----------
`-u` | `--unit` | Generates a Unit test (defaults to generating Feature tests)
`-p` | `--pest` | Generates a Pest PHP test (defaults to generating PHPUnit tests)
`-f` | `--force` | Overwrites existing files with generated files
n/a | `--uninspiring` | Disables inspirational quotes
