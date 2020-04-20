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

It's usually done in the `routes/web.php` and `routes/api.php` files that are [loaded by framework](https://github.com/laravel/laravel/blob/master/app/Providers/RouteServiceProvider.php). The `App/Http/Controllers` namespace is added to the controller names by default.

Read more: https://laravel.com/docs/master/routing

## Security

### Why is [logout button](https://github.com/laravel/ui/blob/ec838c75ba1886d014c5465b1ecc79b2071f46c7/src/Auth/bootstrap-stubs/layouts/app.stub#L58) in the [UI scaffolding](https://laravel.com/docs/master/frontend) submitting a form?

To safeguard you against [cross site request forgery](https://laravel.com/docs/master/csrf). To understand it, let us consider the case without this. Your logout button might be just

```
<a href={{route('logout')}}>Logout</a>
```

and the route would be set up to accept a GET request:

```
Route::get('logout', 'Auth\LoginController@logout')->name('logout');  // or `any` instead of `get`
```

In that case anyone can send your user a link to `https://your-app.example.com/logout` and the user will get logged out when visiting the link. And instead of giving obvious link, one might, for example, give them a shortened URL.

To prevent such scenarios all state changing operations should use appropriate request verbs (not-GET) and Laravel will by default make sure the request contains the CSRF token which makes sure the request was actually initiated from your site.

### How do you prevent SQL injections?

[Query builder](https://laravel.com/docs/master/queries) and [Eloquent](https://laravel.com/docs/master/eloquent) (that's built on top of query builder) both do as much as possible to protect you. You are quite safe as long as you are doing simple queries:

```
User::where('active', true)
    ->whereHas('role', function($role) {
        $role->where('name', 'client');
    });
```

If you have to use raw queries, make sure you are not inserting the user input directly in the query but supply it separately, as you would do with prepared statements:

```
User::whereRaw("(CONCAT(first_name,' ',last_name) like ?)", [$nameToSearch]);
```

## Internals

### How does rate limiting work?

As the first hit comes in, [Rate Limiter](https://github.com/laravel/framework/blob/7.x/src/Illuminate/Cache/RateLimiter.php) stores an entry with value `1` in the cache. The key is the hash of either user identifier or IP with an optional prefix (throttling group). On following hits the value is increased until the decay time passes and the cache entry (*bucket*) is forgotten. Then it starts over.
