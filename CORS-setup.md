# Adding CORS to API
Require the fruitcake/laravel-cors package in your composer.json and update your dependencies
```
composer require fruitcake/laravel-cors
```
### Global usage

To allow CORS for all your routes, add the HandleCors middleware in the $middleware property of **app/Http/Kernel.php** class
```
protected $middleware = [
    // ...
    \Fruitcake\Cors\HandleCors::class,
];
```
### Configuration
```
php artisan vendor:publish --tag="cors"
```
Now update the config to define the paths you want to run the CORS service on, (see Configuration below)
   
**config/cors.php**
```
'paths' => ['api/*'],
```
More details https://github.com/fruitcake/laravel-cors
