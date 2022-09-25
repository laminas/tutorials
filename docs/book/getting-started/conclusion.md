# Conclusion

This concludes our brief look at building a simple, but fully functional,
Laminas laminas-mvc application.

In this tutorial we but briefly touched quite a number of different parts of the
framework.

The most important part of applications built with laminas-mvc are the
[modules](https://docs.laminas.dev/laminas-modulemanager/intro/), the
building blocks of any [laminas-mvc application](https://docs.laminas.dev/laminas-mvc/quick-start/).

To ease the work with dependencies inside our applications, we use the
[service manager](https://docs.laminas.dev/laminas-servicemanager/quick-start/).

To be able to map a request to controllers and their actions, we use
[routes](https://docs.laminas.dev/laminas-router/routing/).

Data persistence was performed using
[laminas-db](https://docs.laminas.dev/laminas-db/adapter/) to communicate with
a relational database. Input data is filtered and validated with [input
filters](https://docs.laminas.dev/laminas-inputfilter/intro/),
and, together with [laminas-form](https://docs.laminas.dev/laminas-form/intro/),
they provide a strong bridge between the domain model and the view layer.

[laminas-view](https://docs.laminas.dev/laminas-view/quick-start/) is
responsible for the View in the MVC stack, together with a vast amount of
[view helpers](https://docs.laminas.dev/laminas-view/helpers/intro/).
