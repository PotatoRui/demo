## About

FILS is a free mobile app for small businesses in the Philippines. Use the app to automate the listing of sales, expenses, debt and track inventory.

The API enables communication between two separate software systems. A software system implementing an API contains functions or subroutines that can be executed by another software system.


## API URL SAMPLE

-   https://fils.otakunity.com/

## API SAMPLE OUTPUT via POSTMAN
```php
$router->mount('/sample', function () use ($router) {
    $router->get('query/{key}', 'SampleController@query');
    $router->post('query/{key}', 'SampleController@query');
});

```

`HTTP VERB: GET`
-   https://fils.otakunity.com/sample/query/09757020264


`HTTP VERB: POST`
-   https://fils.otakunity.com/sample/query/09757020264
