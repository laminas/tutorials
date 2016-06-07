# Setting up a database adapter

> ## TODO
>
> Completely rewrite this to only demonstrate:
>
> - Using factories to inject adapters into controllers, etc.
>   - Demonstrating configuration of the default adapter
>   - Demonstrating retrieval of the default adapter for purposes of injection
> - Using the abstract factory to create named adapters
>   - Demonstrating configuration of named adapters
>   - Demonstrating retrieval of named adapters for purposes of injection

In most cases, e.g. in your controllers, your database adapter can be fetched
directly from the service manager. Some classes however, like
`Zend\Validator\DbRecordExists`, are not aware of the service manager, but still
need an adapter to function.

There are many different ways to provide this functionality to your application.
Below are a few examples.

## Basic setup

Normally you will setup your database adapter using a factory in the service
manager in your configuration. It might look something like this:

```php
// config/autoload/global.php
use Zend\Db\Adapter\Adapter;
use Zend\Db\Adapter\AdapterServiceFactory;

return [
    'db' => [
        'driver' => 'Pdo',
        'dsn'    => 'mysql:dbname=zf2tutorial;host=localhost',
    ],
    'service_manager' => [
        'factories' => [
            Adapter::class => AdapterServiceFactory::class,
        ],
    ],
];
```

The adapter can then be accessed in any ServiceLocatorAware classes.

```php
public function getAdapter()
{
    if (! $this->adapter) {
        $sm = $this->getServiceLocator();
        $this->adapter = $sm->get('Zend\Db\Adapter\Adapter');
    }
    return $this->adapter;
}
```

More information on adapter options can be found in the docs for
[Zend\\Db\\Adapter](http://zendframework.github.io/zend-db/adapter/#creating-an-adapter-using-dependency-injection).

# Setting a static adapter

In order to utilize this adapter in non-ServiceLocatorAware classes, you can use
`Zend\Db\TableGateway\Feature\GlobalAdapterFeature::setStaticAdapter()` to set a
static adapter:

```php
// config/autoload/global.php
use Zend\Db\Adapter\Adapter;
use Zend\Db\Adapter\AdapterServiceFactory;
use Zend\Db\TableGateway\Feature\GlobalAdapterFeature;

return [
    'db' => [
        'driver'         => 'Pdo',
        'dsn'            => 'mysql:dbname=zf2tutorial;host=localhost',
    ],
    'service_manager' => [
        'factories' => [
            Adapter::class => function ($serviceManager) {
                $adapterFactory = new AdapterServiceFactory();
                $adapter = $adapterFactory->createService($serviceManager);
                GlobalAdapterFeature::setStaticAdapter($adapter);
                return $adapter;
            }
        ],
    ],
];
```

The adapter can then later be fetched using
`Zend\Db\TableGateway\Feature\GlobalAdapterFeature::getStaticAdapter()` for use
in e.g. `Zend\Validator\DbRecordExists`:

```php
use Zend\Db\TableGateway\Feature\GlobalAdapterFeature;
use Zend\Validator\Db\RecordExists;

$validator = new RecordExists([
    'table'   => 'users',
    'field'   => 'emailaddress',
    'adapter' => GlobalAdapterFeature::getStaticAdapter()
]);
```
