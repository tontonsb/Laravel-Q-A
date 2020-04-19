# Laravel Q&A

Bits of Laravel knowledge in Q & A format. A.k.a. Laravel "interview" questions.

Please read the [contributing guidelines](CONTRIBUTING.md) if you are interested in sharing your knowledge.

## Trivia

### What does the name *Laravel* mean?

It is [allegedly](https://twitter.com/abigailotwell/status/636178413523329024) inspired by Cair Paravel, the capital of Narnia. Some have also noticed Tay**lor**-**a**-Ot**well**.

## Usage

### How do you route in Laravel?

Specify the route and action using the `Route` facade. There's a variety of ways:

```
Route::get('get-nothing', function() {return 'You get no cabbage.';});
Route::get('cabbages', 'CellarController@getCabbages');
Route::put('lettuce', 'CellarController@putLettuce');
```

It's usually done in the `routes/web.php` and `routes/app.php` files that are [loaded by framework](https://github.com/laravel/laravel/blob/master/app/Providers/RouteServiceProvider.php). The `App/Http/Controllers` namespace is added to the controller names by default.

Read more: https://laravel.com/docs/master/routing

## Internals

### How does rate limiting work?

As the first hit comes in, [Rate Limiter](https://github.com/laravel/framework/blob/7.x/src/Illuminate/Cache/RateLimiter.php) stores an entry with value `1` in the cache. The key is the hash of either user identifier or IP with an optional prefix (cache group). On following hits the value is increased until the decay time passes and the entry is forgotten. Then it starts over.
