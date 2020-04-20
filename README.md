# Laravel Q&A

Bits of Laravel knowledge in Q & A format. A.k.a. Laravel "interview" questions.

The list is not intended to reflect what actually goes on in interviews. Instead the goal is to list questions that might (except for trivia) and should (if they are relevant to job) be asked in interviews — such questions that reflect knowledge and understanding of Laravel instead of plain fact reciting.

Please read the [contributing guidelines](CONTRIBUTING.md) if you are interested in sharing your knowledge.

- [Trivia](#trivia)
  - [What does the name *Laravel* mean?](#what-does-the-name-laravel-mean)
- [General usage](#general-usage)
  - [How do you route in Laravel?](#how-do-you-route-in-laravel)
  - [How do you assign PHP variables to JavaScript variables in the view?](#how-do-you-assign-php-variables-to-javascript-variables-in-the-view)
- [Security](#security)
  - [Why is logout button in the UI scaffolding submitting a form?](#why-is-logout-button-in-the-ui-scaffolding-submitting-a-form)
  - [How do you prevent SQL injections?](#how-do-you-prevent-sql-injections)
- [Specific cases](#specific-cases)
  - [How would you serve a two gigabyte file from the storage?](#how-would-you-serve-a-two-gigabyte-file-from-the-storage)
- [Internals](#internals)
  - [How does rate limiting work?](#how-does-rate-limiting-work)

## Trivia

### What does the name *Laravel* mean?

It is [allegedly](https://twitter.com/abigailotwell/status/636178413523329024) inspired by Cair Paravel, the capital of Narnia. Some have also noticed Tay**lor**-**a**-Ot**well**.

## General usage

### How do you route in Laravel?

Specify the route and action using the `Route` facade. There's a variety of ways:

```php
Route::get('get-nothing', function() {return 'You get no cabbage.';});
Route::get('cabbages', 'CellarController@getCabbages');
Route::put('lettuce', 'CellarController@putLettuce');
```

It's usually done in the `routes/web.php` and `routes/api.php` files that are [loaded by framework](https://github.com/laravel/laravel/blob/master/app/Providers/RouteServiceProvider.php). The `App/Http/Controllers` namespace is added to the controller names by default.

Read more: https://laravel.com/docs/master/routing

### How do you assign PHP variables to JavaScript variables in the view?

There is the `@json` [Blade](https://laravel.com/docs/master/blade) directive that simplifies this as much as humanly possible:

```js
const enemies = @json($enemies)
```


This directive is a [simple wrapper](https://github.com/laravel/framework/blob/0b12ef19623c40e22eff91a4b48cb13b3b415b25/src/Illuminate/View/Compilers/Concerns/CompilesJson.php) around (json_encode)[https://www.php.net/manual/en/function.json-encode.php] and it can handle the same parameters.

It also helps with passing value as attribute to a Vue component, however you must make sure to the attribute value quotes:

```html
<enemy-list 
    :enemies='@json($enemies)'
    :friends='@json($friends)' >
</enemy-list>
```

## Security

### Why is [logout button](https://github.com/laravel/ui/blob/ec838c75ba1886d014c5465b1ecc79b2071f46c7/src/Auth/bootstrap-stubs/layouts/app.stub#L58) in the [UI scaffolding](https://laravel.com/docs/master/frontend) submitting a form?

To safeguard you against [cross site request forgery](https://laravel.com/docs/master/csrf). To understand it, let us consider the case without this. Your logout button might be just

```html
<a href={{route('logout')}}>Logout</a>
```

and the route would be set up to accept a GET request:

```php
Route::get('logout', 'Auth\LoginController@logout')->name('logout');  // or `any` instead of `get`
```

In that case anyone can send your user a link to `https://your-app.example.com/logout` and the user will get logged out when visiting the link. And instead of giving obvious link, one might, for example, give them a shortened URL.

To prevent such scenarios all state changing operations should use appropriate request verbs (not-GET) and Laravel will by default make sure the request contains the CSRF token which makes sure the request was actually initiated from your site.

### How do you prevent SQL injections?

[Query builder](https://laravel.com/docs/master/queries) and [Eloquent](https://laravel.com/docs/master/eloquent) (that's built on top of query builder) both do as much as possible to protect you. You are quite safe as long as you are doing simple queries:

```php
User::where('active', true)
    ->whereHas('role', function($role) {
        $role->where('name', 'client');
    });
```

If you have to use raw queries, make sure you are not inserting the user input directly in the query but supply it separately, as you would do with prepared statements:

```php
User::whereRaw("(CONCAT(first_name,' ',last_name) like ?)", [$nameToSearch]);
```

## Specific cases

### How would you serve a two gigabyte file from the storage?

You should not involve PHP in serving assets if you don't need to. But sometimes you have to control the access. Here are some things that one should not forget when serving huge files:

```php
// In this example Upload is a model with `name`, `path` and `mime` attributes
public function download(Upload $file)
{
    // Disable timeouts
    set_time_limit(0);

    // Grab a stream handle from your storage
    $fs = Storage::getDriver();
    $stream = $fs->readStream($file->path);

    // Disable output buffering or your script will run out of memory
    if (ob_get_level()) ob_end_clean();

    // Return StreamedResponse
    return response()->stream(function() use($stream) {
        fpassthru($stream);
    }, 200, [   // We need to set headers manually
        'Content-Type' => $file->mime,
        'Content-Disposition' => 'attachment; filename="'.$file->name.'"',
    ]);
}
```

There is also another approach — you can instruct your web server to serve the file using features like like [XSendFile](https://tn123.org/mod_xsendfile/) or [X-Accel](https://www.nginx.com/resources/wiki/start/topics/examples/xsendfile/).

## Internals

### How does rate limiting work?

As the first hit comes in, [Rate Limiter](https://github.com/laravel/framework/blob/7.x/src/Illuminate/Cache/RateLimiter.php) stores an entry with value `1` in the cache. The key is the hash of either user identifier or IP with an optional prefix (throttling group). On following hits the value is increased until the decay time passes and the cache entry (*bucket*) is forgotten. Then it starts over.
