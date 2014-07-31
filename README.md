# DunglasAngularCsrfBundle

This [Symfony 2](http://symfony.com) bundle provides automatic [Cross Site Request Forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF or XSRF) protection for client-side [AngularJS](http://angularjs.org/) applications.
It can also be used to secure apps using jQuery or raw JavaScript issuing [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest).

[![Build Status](https://travis-ci.org/dunglas/DunglasAngularCsrfBundle.png?branch=master)](https://travis-ci.org/dunglas/DunglasAngularCsrfBundle)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/4a1e438f-038e-4cd7-ab6e-8849c4586a08/mini.png)](https://insight.sensiolabs.com/projects/4a1e438f-038e-4cd7-ab6e-8849c4586a08)

## How it works

AngularJS' `ng.$http` service has [a built-in CSRF protection system](http://docs.angularjs.org/api/ng.$http#description_security-considerations_cross-site-request-forgery-protection).
To enable it, the server-side application (the Symfony app) must set a cookie containing a XSRF token on the first HTTP request.
Subsequent XHR requests made by AngularJS will provide a special HTTP header containing the value of the cookie.

To prevent CSRF attacks, the server-side application must check that the header's value match the cookie's value.

This bundle provides a [Symfony's Event Listener](http://symfony.com/doc/current/cookbook/service_container/event_listener.html) that set the cookie and another one that checks the HTTP header to block CSRF attacks.
Thanks to DunglasAngularCsrfBundle, you get CSRF security without modifying your code base.

This bundle works fine with [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle).

## Installation

Use [Composer](http://getcomposer.org/) to install this bundle:

    composer require dunglas/angular-csrf-bundle

Add the bundle in your application kernel:

```php
// app/AppKernel.php

public function registerBundles()
{
    return array(
        // ...
        new Dunglas\AngularCsrfBundle\DunglasAngularCsrfBundle(),
        // ...
    );
}
```

Configure URLs where the cookie must be set and that must be protected against CSRF attacks:

```yaml
# app/config/security.yml

dunglas_angular_csrf:
  # Collection of patterns where to set the cookie
  cookie:
      set_on:
          - { path: ^/$ }
          - { route: ^app_, methods: [GET, HEAD] }
  # Collection of patterns to secure
  secure:
    - { path: ^/api, methods: [POST, PUT, PATCH, LINK] }
    - { route: ^api_v2_ }
```

Your Symfony/AngularJS app is now secured.

## Examples

* [DunglasTodoMVCBundle](https://github.com/dunglas/DunglasTodoMVCBundle): an implementation of the TodoMVC app using Symfony, Backbone.js and Chaplin.js

### Usage with AngularJS

To set the $http.defaults.headers on application load, add a run block:

```javascript

App.run(["$http", "$cookies" function($http, $cookies) {
    $http.defaults.headers['X-XSRF-TOKEN'] = $cookies['XSRF-TOKEN'];
}]);
```

Note: You need to use the ngCookies service (https://docs.angularjs.org/api/ngCookies/service/$cookies) for this example to work.

## Full configuration

```yaml
dunglas_angular_csrf:
  token:
      # The CSRF token id
      id: angular
  header:
      # The name of the HTTP header to check (default to the AngularJS default)
      name: X-XSRF-TOKEN
  cookie:
      # The name of the cookie to set (default to the AngularJS default)
      name: XSRF-TOKEN
      # Expiration time of the cookie
      expire: 0
      # Path of the cookie
      path: /
      # Domain of the cookie
      domain: ~
      # If true, set the cookie only on HTTPS connection
      secure: false
      # Patterns of URLs to set the cookie
      set_on:
          - { path: "^/url-pattern", route: "^route_name_pattern$", methods: [GET, POST] }
  # Patterns of URLs to check for a valid CSRF token
  secure:
      - { path: "^/url-pattern", route: "^route_name_pattern$", methods: [GET, POST] }
```

## Credits

This bundle has been written by [Kévin Dunglas](http://dunglas.fr).
