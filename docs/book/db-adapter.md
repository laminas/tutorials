# Setting up a Database Adapter

laminas-db provides a general purpose database abstraction layer. At its heart is
the `Adapter`, which abstracts common database operations across the variety of
drivers we support.

In this guide, we will document how to configure both a single, default adapter
as well as multiple adapters (which may be useful in architectures that have a
cluster of read-only replicated servers and a single writable server of record).

## Installing laminas-db

First, install laminas-db using Composer:

```bash
$ composer require laminas/laminas-db
```

### Installation and automated Configuration

If you are using [laminas-component-installer](https://docs.laminas.dev/laminas-component-installer/)
(installed by default with the skeleton application, and optionally for
Mezzio applications), you will be prompted to install the package
configuration.

- For laminas-mvc applications, choose either `application.config.php` or
  `modules.config.php`.
- For Mezzio applications, choose `config/config.php`.

### Installation and manual Configuration

If you are not using the installer, you will need to manually configure add the
component to your application.

#### Configuration for a laminas-mvc-based Application

For laminas-mvc applications, update your list of modules in either
`config/application.config.php` or `config/modules.config.php` to add an
entry for `'Laminas\Db'` at the top of the list:
  
```php
// In config/modules.config.php
return [
    'Laminas\Db', // <-- This line
    'Laminas\Form', 
    /* ... */
];

// OR in config/application.config.php
return [
    /* ... */
    // Retrieve list of modules used in this application.
    'modules' => [
        'Laminas\Db', // <-- This line
        'Laminas\Form', 
        /* ... */
    ],
    /* ... */
];
```

#### Configuration for a mezzio-based Application

For Mezzio applications, create a new file,
`config/autoload/laminas-db.global.php`, with the following contents:

```php
use Laminas\Db\ConfigProvider;

return (new ConfigProvider())();
```

## Configuring the default Adapter

Within your service factories, you may retrieve the default adapter from your application container using the
class name `Laminas\Db\Adapter\AdapterInterface`:

```php
use Laminas\Db\Adapter\AdapterInterface;

function ($container) {
    return new SomeServiceObject($container->get(AdapterInterface::class));
}
```

When installed and configured, the factory associated with `AdapterInterface`
will look for a top-level `db` key in the configuration, and use it to create an
adapter. As an example, the following would connect to a MySQL database using
PDO, and the supplied PDO DSN:

```php
// In config/autoload/global.php
return [
    'db' => [
        'driver' => 'Pdo',
        'dsn'    => 'mysql:dbname=laminastutorial;host=localhost;charset=utf8',
    ],
];
```

More information on adapter configuration can be found in the docs for
[Laminas\\Db\\Adapter](http://docs.laminas.dev/laminas-db/adapter/#creating-an-adapter-using-dependency-injection).

## Configuring named Adapters

Sometimes you may need multiple adapters. As an example, if you work with a
cluster of databases, one may allow write operations, while another may be
read-only.

laminas-db provides an [abstract factory](https://docs.laminas.dev/laminas-servicemanager/configuring-the-service-manager/#abstract-factories),
`Laminas\Db\Adapter\AdapterAbstractServiceFactory`, for this purpose. To use it,
you will need to create named configuration keys under `db.adapters`, each with
configuration for an adapter:

```php
// In config/autoload/global.php
return [
    'db' => [
        'adapters' => [
            'Application\Db\WriteAdapter' => [
                'driver' => 'Pdo',
                'dsn'    => 'mysql:dbname=application;host=canonical.example.com;charset=utf8',
            ],
            'Application\Db\ReadOnlyAdapter' => [
                'driver' => 'Pdo',
                'dsn'    => 'mysql:dbname=application;host=replica.example.com;charset=utf8',
            ],
        ],
    ],
];
```

You retrieve the database adapters using the keys you define, so ensure they are
unique to your application, and descriptive of their purpose!

### Retrieving named Adapters

Retrieve named adapters in your service factories just as you would another
service:

```php
function ($container) {
    return new SomeServiceObject($container->get('Application\Db\ReadOnlyAdapter'));
}
```

### Using the `AdapterAbstractServiceFactory` as a Factory

Depending on what application container you use, abstract factories may not be
available. Alternately, you may want to reduce lookup time when retrieving an
adapter from the container (abstract factories are consulted last!).
laminas-servicemanager abstract factories work as factories in their own right, and
are passed the service name as an argument, allowing them to vary their return
value based on requested service name. As such, you can add the following
service configuration as well:

```php
use Laminas\Db\Adapter\AdapterAbstractServiceFactory;

// If using laminas-mvc:
// In module/YourModule/config/module.config.php
'service_manager' => [
    'factories' => [
        'Application\Db\WriteAdapter' => AdapterAbstractServiceFactory::class,
    ],
],

// If using Mezzio
'dependencies' => [
    'factories' => [
        'Application\Db\WriteAdapter' => AdapterAbstractServiceFactory::class,
    ],
],
```
