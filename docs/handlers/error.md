---
title: System Error Handler
---

Things go wrong. You can't predict errors, but you can anticipate them. Each Slim Framework application has an error handler that receives all uncaught PHP exceptions. This error handler also receives the current HTTP request and response objects, too. The error handler must prepare and return an appropriate Response object to be returned to the HTTP client.

## Default error handler

The default error handler is very basic. It sets the Response status code to `500`, it sets the Response content type to `text/html`, and appends a generic error message into the Response body.

This is _probably_ not appropriate for production applications. You are strongly encouraged to implement your own Slim application error handler.

The default error handler can also include detailed error diagnostic information. To enable this you need to set the `displayErrorDetails` setting to true:

{% highlight php %}
$configuration = [
    'settings' => [
        'displayErrorDetails' => true,
    ],
];
$c = new \Slim\Container($configuration);
$app = new \Slim\App($c);
{% endhighlight %}

## Custom error handler

A Slim Framework application's error handler is a Pimple service. You can substitute your own error handler by defining a custom Pimple factory method with the application container.

There are two ways to inject handlers:

### Pre App

{% highlight php %}
$c = new \Slim\Container();
$c['errorHandler'] = function ($c) {
    return function ($request, $response, $exception) use ($c) {
        return $c['response']->withStatus(500)
                             ->withHeader('Content-Type', 'text/html')
                             ->write('Something went wrong!');
    };
};
$app = new \Slim\App($c);
{% endhighlight %}

### Post App

{% highlight php %}
$app = new \Slim\App();
$c = $app->getContainer();
$c['errorHandler'] = function ($c) {
    return function ($request, $response, $exception) use ($c) {
        return $c['response']->withStatus(500)
                             ->withHeader('Content-Type', 'text/html')
                             ->write('Something went wrong!');
    };
};
{% endhighlight %}

In this example, we define a new `errorHandler` factory that returns a callable. The returned callable accepts three arguments:

1. A `\Psr\Http\Message\ServerRequestInterface` instance
2. A `\Psr\Http\Message\ResponseInterface` instance
3. A `\Exception` instance

The callable **MUST** return a new `\Psr\Http\Message\ResponseInterface` instance as is appropriate for the given exception.

### Disabling

To completely disable Slim's error handling, simply remove the error handler from the container:

{% highlight php %}
unset($app->getContainer()['errorHandler']);
{% endhighlight %}

You are now responsible for handling any exceptions that occur in your application as they will not be handled by Slim.