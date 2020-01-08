# Migration from Zend Framework, Apigility, or Expressive

Laminas and its subprojects are an open source continuation of [Zend
Framework](https://github.com/zendframework) and its subprojects, and are the
official fork of the project now that it has been archived and abandoned.

> For purposes of this document, when we refer to "Zend Framework", we mean any
> package originally provided by the Zend Framework project or its subprojects,
> **including those provided by [Apigility](https://apigility.org) or
> [Expressive](https://getexpressive.org).**

You can migrate:

- Individual projects or applications that depend on Zend Framework, or have
  dependencies on components that depend on Zend Framework.

- Libraries that depend on Zend Framework components.

## 0. Preparation

### Ensure you have an up-to-date Composer

Due to features of [Composer](https://getcomposer.org) our dependency plugin
uses, we require Composer 1.7.0 and up. If you're unsure what version you are
on, run `composer --version`. If you are on an older version, run composer
self-update.

(We assume you use Composer to install Zend Framework, as it is the only
supported mechanism currently.)

### Make sure your code is under version control

The migration tool changes source code, updates tempaltes, modifies your
`composer.json` and removes your `composer.lock` and `vendor/` subdirectory,
among other things. If you want to be able to roll back in the case of a
problem, please make certain your code is under version control!

We recommend [Git](https://git-scm.org). If your application is not currently
under version control, install git, and then:

```bash
$ cd path/to/your/application
$ git init .
$ echo "vendor/" > .gitignore
$ git add .
$ git commit -m 'Initial import'
```

You will then be able to see what changes you introduce.

## 1. Install laminas-migration

There are three ways to install the tool:

1. As a local dependency
1. Via a global composer requirement
1. Via cloning

### 1. As a local dependency

In this case require it as you would any other dependency:

```bash
$ composer require laminas/laminas-migration
```

For purposes of usage, when we refer to the `laminas-migration` CLI tool, the
path will be `./vendor/bin/laminas-migration`.

### 2. Via a global composer requirement

Composer allows you to require packages in a _global_ context as well. If you
choose this option, you will run:

```bash
$ composer global require laminas/laminas-migration
```

It is up to you to ensure that the `vendor/bin/` subdirectory of your global
Composer installation is in your environment `$PATH`. You can locate the global
context directory using:

```bash
$ composer global config home
```

> #### Adding to the PATH
>
> The mechanism for adding to your environment `$PATH` variable depends on your
> operating system.
>
> For Linux, Mac, and other *nix variants, you can do so by adding a line like
> the following at the end of your profile configuration file (e.g., `$HOME/.bashrc`,
> `$HOME/.zshrc`, `$HOME/.profile`, etc.):
>
> ```bash
> export PATH={path to add}:$PATH
> ```
>
> For Windows, the situation is a bit more involved; [this HOWTO](https://www.architectryan.com/2018/03/17/add-to-the-path-on-windows-10/)
> provides a good tutorial on the subject.

### 3. Via cloning

Clone the repository somewhere:

```bash
$ git clone https://github.com/laminas/laminas-migration.git
```

Install dependencies:

```bash
$ cd laminas-migration
$ composer install
```

From there, either add the `bin/` directory to your `$PATH` (see the [note on
adding to the PATH](#adding-to-the-path), above), symlink the
`bin/laminas-migration` script to a directory in your `$PATH`, or create an
alias to the `bin/laminas-migration` script using your shell:

```bash
# Adding to PATH:
$ export PATH=/path/to/laminas-migration/bin:$PATH
# Symlinking to a directory in your PATH:
$ cd $HOME/bin && ln -s /path/to/laminas-migration/bin/laminas-migration .
# creating an alias:
$ alias laminas-migration=/path/to/laminas-migration/bin/laminas-migration
```

## 2. Run the migration command

Once you have installed the laminas-migration tooling, enter a project you wish
to migrate, and run the following:

```bash
$ laminas-migration migrate
```

You may want to use the `--exclude` or `-e` option one or more times for
directories to _exclude_ from the rewrite, or the `--filter` or `-f` option one or
more times to provide regular expressions of which files to _include_ in the
rewrite.

As an example, to exclude the `data/` subdirectory, you could run:

```bash
$ laminas-migration migrate -e data
```

## 3. (Optional) Verify changes

At this point, you can verify that the changes look correct. As an example, if
you use git, you could run the following command to determine what has changed:

```bash
$ git diff
```

Things you might want to look for:

- Renaming of files or classes in your own code that reference ZF components. As
  an example `My\Models\ZendMailTransport` might get renamed to
  `My\Models\LaminasMailTransport`. Such renames are typically okay, and any
  references to those classes will be rewritten as well. However, if external
  code may depend on them, you should verify.

- Changes to configuration keys. Most of these are made to match changes in the
  Laminas libraries themselves, but you should check to see if they are specific
  to your own code.

## 4. Install dependencies

Once migration is done and you've performed any verification steps you need,
install dependencies:

```bash
$ composer install
```

### 5. Test

Run your unit tests, do end-to-end tests, whatever — but exercise the
application in some way.

If anything does not work, determine the specifics, and report either:

- In the repository of the specific component where you are observing problems.
- The #laminas-issues channel of the [Laminas Slack](https://laminas.dev/chat/).

## Summary

To summarize the steps to test:

```bash
$ composer global require laminas/laminas-migration
$ cd some/project
$ laminas-migration migrate -e data
$ composer config repositories.laminas composer https://laminas.mwop.net/repo/testing
$ composer install
```

## FAQ

### Module and Config Post Processor injection

If you are migrating an MVC, Apigility, or Expressive application to Laminas,
the migration tooling attempts to inject some code in your application. This can
fail if you have non-standard configuration.

#### Migrating MVC and Apigility applications

When migrating MVC and Apigility applications to Laminas MVC and Laminas API
Tools, the migration tooling attempts to add `Laminas\ZendFrameworkBridge` as a
module to the top of the `config/modules.config.php` file. If injection fails,
add the module in a way appropriate to your application.

#### Expressive

When migrating Expressive applications to Mezzio, the migration tooling attempts
to add `Laminas\ZendFrameworkBridge\ConfigPostProcessor` as a post processor
class to the `ConfigAggregator` constructor. The `ConfigAggregator` constructor
has the following signature:

```php
public function __construct(
    array $providers = [],
    ?string $cachedConfigFile = null,
    array $postProcessors = []
)
```

Typically, the structure of the `config/config.php` file in an Expressive and
Mezzio applications looks like the following:

```php
$cacheConfig = [
    'config_cache_path' => 'data/cache/app_config.php',
];

$aggregator = new ConfigAggregator([
    // config providers from 3rd party code
    // ...

    // App-specific modules
    // ...

    // Include cache configuration
    new ArrayProvider($cacheConfig),

    // Load application config in a pre-defined order in such a way that local settings
    // overwrite global settings. (Loaded as first to last):
    //   - `global.php`
    //   - `*.global.php`
    //   - `local.php`
    //   - `*.local.php`
    new PhpFileProvider('config/autoload/{{,*.}global,{,*.}local}.php'),

    // Load development config if it exists
    new PhpFileProvider('config/development.config.php'),
], $cacheConfig['config_cache_path']);

return $aggregator->getMergedConfig();
```

As such, the migration tooling rewrites the second to last line to read:

```php
], $cacheConfig['config_cache_path'], [\Laminas\ZendFrameworkBridge\ConfigPostProcessor::class]);
```

In most cases, failure to inject means that the individual arguments have
been pushed to their own line. In such cases, add the third argument as
detailed above.

In other cases, applications may already be using post processors. If so,
add `\Laminas\ZendFrameworkBridge\ConfigPostProcessor::class` to the list of
post processors.

### Clear your caches

If your application is not running in development mode, you will need to clear
any configuration caches you have before testing. If you are using
zf-development-mode (which becomes laminas-development-mode!), try enabling
development mode:

```bash
$ composer development-enable
```

Expressive/Mezzio users can use the `clear-config-cache` command:

```bash
$ composer clear-config-cache
```
