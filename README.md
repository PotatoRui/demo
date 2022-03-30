## Prerequisites

-   PHP 8.0 or higher

## Routing

Hook **routes** (a combination of one or more HTTP methods and a pattern) using `$router->match(method(s), pattern, function)`:

```php
$router->match('GET|POST', 'pattern', function() { … });
```

Router supports `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, and `OPTIONS` HTTP request methods. Pass in a single request method, or multiple request methods separated by `|`.

When a route matches against the current URL (e.g. `$_SERVER['REQUEST_URI']`), the attached **route handling function** will be executed. The route handling function must be a [callable](http://php.net/manual/en/language.types.callable.php). Only the first route matched will be handled. When no matching route is found, a 404 handler will be executed.

### Routing Shorthands

Shorthands for single request methods are provided:

```php
$router->get('pattern', function() { /* ... */ });
$router->post('pattern', function() { /* ... */ });
$router->put('pattern', function() { /* ... */ });
$router->delete('pattern', function() { /* ... */ });
$router->options('pattern', function() { /* ... */ });
$router->patch('pattern', function() { /* ... */ });
```

You can use this shorthand for a route that can be accessed using any method:

```php
$router->all('pattern', function() { /* ... */ });
```

Route Patterns can be static or dynamic:

-   **Static Route Patterns** contain no dynamic parts and must match exactly against the `path` part of the current URL.
-   **Dynamic Route Patterns** contain dynamic parts that can vary per request. The varying parts are named **subpatterns** and are defined using either Perl-compatible regular expressions (PCRE) or by using **placeholders**

#### Static Route Patterns

A static route pattern is a regular string representing a URI. It will be compared directly against the `path` part of the current URL.

Examples:

-   `/about`
-   `/contact`

Usage Examples:

```php
// This route handling function will only be executed when visiting '/anime'
$router->get('/users', function() {
    echo 'Anime Page Contents';
});
```

#### Dynamic PCRE-based Route Patterns

This type of Route Patterns contain dynamic parts which can vary per request. The varying parts are named **subpatterns** and are defined using regular expressions.

Examples:

-   `/users/(\d+)`
-   `/profile/(\w+)`

Commonly used PCRE-based subpatterns within Dynamic Route Patterns are:

-   `\d+` = One or more digits (0-9)
-   `\w+` = One or more word characters (a-z 0-9 \_)
-   `[a-z0-9_-]+` = One or more word characters (a-z 0-9 \_) and the dash (-)
-   `.*` = Any character (including `/`), zero or more
-   `[^/]+` = Any character but `/`, one or more

Note: The [PHP PCRE Cheat Sheet](https://courses.cs.washington.edu/courses/cse154/15sp/cheat-sheets/php-regex-cheat-sheet.pdf) might come in handy.

The **subpatterns** defined in Dynamic PCRE-based Route Patterns are converted to parameters which are passed into the route handling function. Prerequisite is that these subpatterns need to be defined as **parenthesized subpatterns**, which means that they should be wrapped between parens:

```php
// Bad Routing
$router->get('/hello/\w+', function($name) {
    echo 'Hello ' . htmlentities($name);
});

// Good Routing
$router->get('/hello/(\w+)', function($name) {
    echo 'Hello ' . htmlentities($name);
});
```

Note: The leading `/` at the very beginning of a route pattern is not mandatory, but is recommended.

When multiple subpatterns are defined, the resulting **route handling parameters** are passed into the route handling function in the order they are defined in:

```php
$router->get('/anime/(\d+)/photos/(\d+)', function($animeId, $photoId) {
    echo 'Anime  #' . $movieId . ', photo #' . $photoId;
});
```

#### Dynamic Placeholder-based Route Patterns

This type of Route Patterns are the same as **Dynamic PCRE-based Route Patterns**, but with one difference: they don't use regexes to do the pattern matching but they use the more easy **placeholders** instead. Placeholders are strings surrounded by curly braces, e.g. `{name}`. You don't need to add parens around placeholders.

Examples:

-   `/anime/{id}`
-   `/profile/{username}`

Placeholders are easier to use than PRCEs, but offer you less control as they internally get translated to a PRCE that matches any character (`.*`).

```php
$router->get('/anime/{animeId}/photos/{photoId}', function($animeId, $photoId) {
    echo 'Anime #' . $animeId . ', photo #' . $photoId;
});
```

Note: the name of the placeholder does not need to match with the name of the parameter that is passed into the route handling function:

```php
$router->get('/anime/{foo}/photos/{bar}', function($animeId, $photoId) {
    echo 'Anime #' . $animeId . ', photo #' . $photoId;
});
```

### Optional Route Subpatterns

Route subpatterns can be made optional by making the subpatterns optional by adding a `?` after them. Think of blog URLs in the form of `/blog(/year)(/month)(/day)(/slug)`:

```php
$router->get(
    '/blog(/\d+)?(/\d+)?(/\d+)?(/[a-z0-9_-]+)?',
    function($year = null, $month = null, $day = null, $slug = null) {
        if (!$year) { echo 'Blog overview'; return; }
        if (!$month) { echo 'Blog year overview'; return; }
        if (!$day) { echo 'Blog month overview'; return; }
        if (!$slug) { echo 'Blog day overview'; return; }
        echo 'Blogpost ' . htmlentities($slug) . ' detail';
    }
);
```

The code snippet above responds to the URLs `/blog`, `/blog/year`, `/blog/year/month`, `/blog/year/month/day`, and `/blog/year/month/day/slug`.

Note: With optional parameters it is important that the leading `/` of the subpatterns is put inside the subpattern itself. Don't forget to set default values for the optional parameters.

The code snipped above unfortunately also responds to URLs like `/blog/foo` and states that the overview needs to be shown - which is incorrect. Optional subpatterns can be made successive by extending the parenthesized subpatterns so that they contain the other optional subpatterns: The pattern should resemble `/blog(/year(/month(/day(/slug))))` instead of the previous `/blog(/year)(/month)(/day)(/slug)`:

```php
$router->get('/blog(/\d+(/\d+(/\d+(/[a-z0-9_-]+)?)?)?)?', function($year = null, $month = null, $day = null, $slug = null) {
    // ...
});
```

Note: It is highly recommended to **always** define successive optional parameters.

To make things complete use [quantifiers](http://www.php.net/manual/en/regexp.reference.repetition.php) to require the correct amount of numbers in the URL:

```php
$router->get('/blog(/\d{4}(/\d{2}(/\d{2}(/[a-z0-9_-]+)?)?)?)?', function($year = null, $month = null, $day = null, $slug = null) {
    // ...
});
```

### Subrouting / Mounting Routes

Use `$router->mount($baseroute, $fn)` to mount a collection of routes onto a subroute pattern. The subroute pattern is prefixed onto all following routes defined in the scope. e.g. Mounting a callback `$fn` onto `/anime` will prefix `/anime` onto all following routes.

```php
$router->mount('/anime', function() use ($router) {

    // will result in '/anime/'
    $router->get('/', function() {
        echo 'anime overview';
    });

    // will result in '/anime/id'
    $router->get('/(\d+)', function($id) {
        echo 'anime id ' . htmlentities($id);
    });

});
```

Nesting of subroutes is possible, just define a second `$router->mount()` in the callable that's already contained within a preceding `$router->mount()`.

### `Class@Method` calls

We can route to the class action like so:

```php
$router->get('/(\d+)', '\App\Controllers\AnimeController@showDetails');
```

When a request matches the specified route URI, the `showDetails` method on the `AnimeController` class will be executed. The defined route parameters will be passed to the class method.

The method can be static (recommended) or non-static (not-recommended). In case of a non-static method, a new instance of the class will be created.

If most/all of your handling classes are in one and the same namespace, you can set the default namespace to use on your router instance via `setNamespace()`

```php
$router->setNamespace('\App\Controllers');
$router->get('/users/(\d+)', 'UserController@showProfile');
```

### Custom 404

The default 404 handler sets a 404 status code and exits. You can override this default 404 handler by using `$router->set404(callable);`

```php
$router->set404(function() {
    header('HTTP/1.1 404 Not Found');
    // ... do something special here
});
```

You can also define multiple custom routes e.x. you want to define an `/api` route, you can print a custom 404 page:

```php
$router->set404('/api(/.*)?', function() {
    header('HTTP/1.1 404 Not Found');
    header('Content-Type: application/json');

    $jsonArray = array();
    $jsonArray['status'] = "404";
    $jsonArray['status_text'] = "route not defined";

    echo json_encode($jsonArray);
});
```

Also supported are `Class@Method` callables:

```php
$router->set404('\App\Controllers\ErrorController@_404');
```

The 404 handler will be executed when no route pattern was matched to the current URL.

💡 You can also manually trigger the 404 handler by calling `$router->trigger404()`

```php
$router->get('/anime/([a-z0-9-]+)', function($id) use ($router) {
    if (!Anime::exists($id)) {
        $router->trigger404();
        return;
    }

    // …
});
```

### Before Route Middlewares

Router supports **Before Route Middlewares**, which are executed before the route handling is processed.

Like route handling functions, you hook a handling function to a combination of one or more HTTP request methods and a specific route pattern.

```php
$router->before('GET|POST', '/admin/.*', function() {
    if (!isset($_SESSION['user'])) {
        header('location: /auth/login');
        exit();
    }
});
```

Unlike route handling functions, more than one before route middleware is executed when more than one route match is found.

### Before Router Middlewares

Before route middlewares are route specific. Using a general route pattern (viz. _all URLs_), they can become **Before Router Middlewares** _(in other projects sometimes referred to as before app middlewares)_ which are always executed, no matter what the requested URL is.

```php
$router->before('GET', '/.*', function() {
    // ... this will always be executed
});
```

### After Router Middleware / Resolve Callback

Run one (1) middleware function, name the **After Router Middleware** _(in other projects sometimes referred to as after app middlewares)_ after the routing was processed. Just pass it along the `$this->router->resolve()` function in `Daishi\Foundation\Application`. The run callback is route independent.

```php
$this->router->resolve(function() { … });
```

Note: If the route handling function has `exit()`ed the resolve callback won't be run.

### Overriding the request method

Use `X-HTTP-Method-Override` to override the HTTP Request Method. Only works when the original Request Method is `POST`. Allowed values for `X-HTTP-Method-Override` are `PUT`, `DELETE`, or `PATCH`.

## A note on working with PUT

There's no such thing as `$_PUT` in PHP. One must fake it:

```php
$router->put('/anime/(\d+)', function($id) {

    // Fake $_PUT
    $_PUT  = array();
    parse_str(file_get_contents('php://input'), $_PUT);

    // ...

});
```

## A note on making HEAD requests

When making `HEAD` requests all output will be buffered to prevent any content trickling into the response body, as defined in [RFC2616 (Hypertext Transfer Protocol -- HTTP/1.1)](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.4):

> The HEAD method is identical to GET except that the server MUST NOT return a message-body in the response. The metainformation contained in the HTTP headers in response to a HEAD request SHOULD be identical to the information sent in response to a GET request. This method can be used for obtaining metainformation about the entity implied by the request without transferring the entity-body itself. This method is often used for testing hypertext links for validity, accessibility, and recent modification.

To achieve this, router will internally re-route `HEAD` requests to their equivalent `GET` request and automatically suppress all output.
