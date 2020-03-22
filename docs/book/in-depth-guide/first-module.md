# Introducing the Blog Module

Now that we know about the basics of the laminas-mvc skeleton application,
let's continue and create our very own module. We will create a module named
"Blog". This module will display a list of database entries that represent a
single blog post. Each post will have three properties: `id`, `text`, and
`title`. We will create forms to enter new posts into our database and to edit
existing posts. Furthermore we will do so by using best-practices throughout the
whole tutorial.

## Writing a new Module

Let's start by creating a new folder under the `/module` directory called
`Blog`, with the following stucture:

```text
module/
    Blog/
        config/
        src/
        view/
```

To be recognized as a module by the [ModuleManager](https://docs.laminas.dev/laminas-modulemanager/intro/),
we need to do three things:

- Tell Composer how to autoload classes from our new module.
- Create a `Module` class in the `Blog` namespace.
- Notify the application of the new module.

Let's tell Composer about our new module. Open the `composer.json` file in the
project root, and edit the `autoload` section to add a new PSR-4 entry for the
`Blog` module; when you're done, it should read:

```json
"autoload": {
   "psr-4": {
        "Application\\": "module/Application/src/",
        "Album\\": "module/Album/src/",
        "Blog\\": "module/Blog/src/"
   }
}
```

Once you're done, tell Composer to update its autoloading definitions:

```bash
$ composer dump-autoload
```

Next, we will create a `Module` class under the `Blog` namespace. Create the
file `module/Blog/src/Module.php` with the following contents:

```php
namespace Blog;

class Module
{
}
```

We now have a module that can be detected by the
[ModuleManager](https://docs.laminas.dev/laminas-modulemanager/intro/).
Let's add this module to our application. Although our module doesn't do
anything yet, just having the `Module.php` class allows it to be loaded by the
ModuleManager.  To do this, add an entry for `Blog` to the modules array inside
`config/modules.config.php`:

```php
// In config/modules.config.php:

return [
    /* ... */
    'Application',
    'Album',
    'Blog',
];
```

If you refresh your application you should see no change at all (but also no
errors).

At this point it's worth taking a step back to discuss what modules are for. In
short, a module is an encapsulated set of features for your application. A
module might add features to the application that you can see, like our `Blog`
module; or it might provide background functionality for other modules in the
application to use, such as interacting with a third party API.

Organizing your code into modules makes it easier for you to reuse functionality
in other applications, or to use modules written by the community.

## Configuring the Module

The next thing we're going to do is add a route to our application so that our
module can be accessed through the URL `localhost:8080/blog`. We do this by
adding router configuration to our module, but first we need to let the
`ModuleManager` know that our module has configuration that it needs to load.

This is done by adding a `getConfig()` method to the `Module` class that
returns the configuration. (This method is defined in the
`ConfigProviderInterface`, although explicitly implementing this interface in the
module class is optional.) This method should return either an `array` or a
`Traversable` object. Continue by editing `module/Blog/src/Module.php`:

```php
// In /module/Blog/src/Module.php:
class Module
{
    public function getConfig() : array
    {
        return [];
    }
}
```

With this, our module is now able to be configured. Configuration files can
become quite big, though, and keeping everything inside the `getConfig()` method
won't be optimal. To help keep our project organized, we're going to put our
array configuration in a separate file. Go ahead and create this file at
`module/Blog/config/module.config.php`:

```php
return [];
```

Now rewrite the `getConfig()` function to include this newly created
file instead of directly returning the array:

```php
// In /module/Blog/src/Module.php:

public function getConfig() : array
{
    return include __DIR__ . '/../config/module.config.php';
}
```

Reload your application and you'll see that nothing changes. Creating,
registering, and adding empty configuration for a new module has no visible
effect on the application. Next we add the new route to our configuration file:

```php
// In /module/Blog/config/module.config.php:
namespace Blog;

use Laminas\Router\Http\Literal;

return [
    // This lines opens the configuration for the RouteManager
    'router' => [
        // Open configuration for all possible routes
        'routes' => [
            // Define a new route called "blog"
            'blog' => [
                // Define a "literal" route type:
                'type' => Literal::class,
                // Configure the route itself
                'options' => [
                    // Listen to "/blog" as uri:
                    'route' => '/blog',
                    // Define default controller and action to be called when
                    // this route is matched
                    'defaults' => [
                        'controller' => Controller\ListController::class,
                        'action'     => 'index',
                    ],
                ],
            ],
        ],
    ],
];
```

We've now created a route called `blog` that listens to the URL
`localhost:8080/blog`. Whenever someone accesses this route, the `indexAction()`
function of the class `Blog\Controller\ListController` will be executed.
However, this controller does not exist yet, so if you reload the page you will
see this error message:

```text
A 404 error occurred
Page not found.
The requested controller could not be mapped by routing.

Controller:
Blog\Controller\ListController(resolves to invalid controller class or alias: Blog\Controller\ListController)
```

We now need to tell our module where to find this controller named
`Blog\Controller\ListController`. To achieve this we have to add this key to the
`controllers` configuration key inside your
`module/Blog/config/module.config.php`.

```php
namespace Blog;

use Laminas\ServiceManager\Factory\InvokableFactory;

return [
    'controllers' => [
        'factories' => [
            Controller\ListController::class => InvokableFactory::class,
        ],
    ],
    /* ... */
];
```

This configuration defines a factory for the controller class
`Blog\Controller\ListController`, using the laminas-servicemanager
`InvokableFactory` (which, internally, instantiates the class with no
arguments).  Reloading the page should then give you:

```text
Fatal error: Class 'Blog\Controller\ListController' not found in {projectPath}/vendor/laminas/laminas-servicemanager/src/Factory/InvokableFactory.php on line 32
```

This error tells us that the application knows what class to load, but was not
able to autoload it. In our case, we've already setup autoloading, but have not
yet defined the controller class!

Create the file `module/Blog/src/Controller/ListController.php` with the
following contents:

```php
namespace Blog\Controller;

class ListController
{
}
```

Reloading the page now will finally result into a new screen. The new error message looks like this:

```html
A 404 error occurred
Page not found.
The requested controller was not dispatchable.

Controller:
Blog\Controller\List(resolves to invalid controller class or alias: Blog\Controller\List)

Additional information:
Laminas\ServiceManager\Exception\InvalidServiceException

File:
{projectPath}/vendor/laminas/laminas-mvc/src/Controller/ControllerManager.php:{lineNumber}

Message:
Plugin of type "Blog\Controller\ListController" is invalid; must implement Laminas\Stdlib\DispatchableInterface
```

This happens because our controller must implement
[DispatchableInterface](https://github.com/laminas/laminas-stdlib/blob/0e66240daec93f19ae104084e369c393453fc901/src/DispatchableInterface.php)
in order to be 'dispatched' (or run) by laminas-mvc. laminas-mvc provides a base
controller implementation of it with
[AbstractActionController](https://github.com/laminas/laminas-mvc/blob/9fc7f2cf569760d6a0cdc62f56e042a0aee3a4c9/src/Controller/AbstractActionController.php),
which we are going to use. Let's modify our controller now:

```php
// In /module/Blog/src/Blog/Controller/ListController.php:

namespace Blog\Controller;

use Laminas\Mvc\Controller\AbstractActionController;

class ListController extends AbstractActionController
{
}
```

It's now time for another refresh of the site. You should now see a new error message:

```text
An error occurred

An error occurred during execution; please try again later.

Additional information:

Laminas\View\Exception\RuntimeException

File:
{projectPath}/vendor/laminas/laminas-view/src/Renderer/PhpRenderer.php:{lineNumber}

Message:
Laminas\View\Renderer\PhpRenderer::render: Unable to render template "blog/list/index"; resolver could not resolve to a file
```

Now the application tells you that a view template-file cannot be rendered,
which is to be expected as we've not created it yet. The application is
expecting it to be at `module/Blog/view/blog/list/index.phtml`. Create this
file and add some dummy content to it:

```html
<!-- Filename: module/Blog/view/blog/list/index.phtml -->
<h1>Blog\Controller\ListController::indexAction()</h1>
```

Before we continue let us quickly take a look at where we placed this file. Note
that view files are found within the `/view` subdirectory, not `/src` as they
are not PHP class files, but template files for rendering HTML. The path,
however, deserves some explanation. First we have the lowercased namespace `blog`,
followed by the lowercased controller name `list` (without the suffix 'controller'),
and lastly comes the name of the action that we are accessing, `index` (again without the
suffix 'action'). As a templated string, you can think of it as:
`view/{namespace}/{controller}/{action}.phtml`. This has become a community
standard but you have the freedom to specify custom paths if desired.

However creating this file alone is not enough and this brings as to the final
topic of this part of the tutorial. We need to let the application know where
to look for view files. We do this within our module's configuration file,
`module.config.php`.

```php
// In module/Blog/config/module.config.php:

return [
    'controllers' => [ /** Controller Configuration */ ],
    'router'      => [ /** Route Configuration */ ],
    'view_manager' => [
        'template_path_stack' => [
            __DIR__ . '/../view',
        ],
    ],
];
```

The above configuration tells the application that the folder
`module/Blog/view/` has view files in it that match the standard path format:
`view/{namespace}/{controller}/{action}.phtml`. It is important to note that the
`view_manager` configuration not only allows you to ship view files for your
module, but also to overwrite view files from other modules.

Reload your site now. Finally we are at a point where we see something different
than an error being displayed! You should see the standard Laminas Skeleton
Application template page with **Blog\Controller\ListController::indexAction()** as the
header.
    
Congratulations, not only have you created a simple "Hello World" style module,
you also learned about many error messages and their causes. If we didn't
exhaust you too much, continue with our tutorial, and let's create a module that
actually does something.
