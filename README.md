# mRcore Framework v5.7

## What Is Mrcore

mRcore is a module system for Laravel allowing you to build all your applications as reusable modules.
Module resemble the Laravel folder structure and can be plugged into a single Laravel instance.
mRcore solves module loading dependency order and in-place live asset handling.  Modules can be
full web UIs, REST APIs and/or full Console command line apps.  A well build module becomes your
shared PHP library, a native API, which can be reused as dependencies in other modules.

We firmly believe that all code should be built as modules and not in Laravels directory structure itself.
Laravel simply becomes the "app server".  A single Laravel instance can host any number of modules. 

See https://github.com/mrcore5/framework for details and installation instructions.



## Installing mRcore Framework

1. Install stock Laravel 5.7
	```
	cd ~/Code/acme
	laravel new laravel57
	cd laravel57
	php artisan serve
	```
	* Visit http://localhost:8000 to see stock laravel!  Then stop the server.

2. Install mRcore Framework on Laravel 5.7
	```
	composer require mrcore/foundation:5.7.*
	php artisan mrcore:foundation:install
	php artisan serve
	```
	* Visit http://localhost:8000 to see mRcore welcome!




## Building Modules

Assume you are a vendor called "acme" with 3 mrcore modules and one laravel57 install.
A good folder structure looks like this
```
~/Code/
  acme/
    laravel57/
    wiki/
    blog/
    dealer-portal/
```

We can create these 3 modules using mRcore framework app builder command
```
./artisan mrcore:foundation:app:make acme/wiki --template=mrcore5-src --path=~/Code/acme/wiki
./artisan mrcore:foundation:app:make acme/blog --template=mrcore5-src --path=~/Code/acme/blog
./artisan mrcore:foundation:app:make acme/dealer-portal --template=mrcore5-src --path=~/Code/acme/dealer-portal
```

For coding convenience, we create a `laravel57/modules` folder and symlink our modules there
```
mkdir -p ~/Code/acme/laravel57/modules/acme
cd ~/Code/acme/laravel57/modules/acme
ln -s ../../../wiki
ln -s ../../../blog
ln -s ../../../dealer-portal
```
Back to `cd ~/Code/acme/laravel57`

Modify `composer.json` and add the `path` type repository pointing to `./modules/*/*` so we can code our packages LIVE!

**Except of top of composer.json file only**
```json
{
  "name": "acme/laravel57",
  "description": "Acme Laravel 5.7",
  "keywords": ["laravel", "acme"],
  "license": "MIT",
  "type": "project",
  "repositories": [
    {"type": "path", "url": "./modules/*/*"}
  ],
  "require": {
    "php": "^7.1.3",
    "fideloper/proxy": "^4.0",
    "laravel/framework": "5.7.*",
    "laravel/tinker": "^1.0",
    "mrcore/foundation": "5.7.*",

    "acme/wiki": "*@dev",
    "acme/blog": "*@dev",
    "acme/dealer-portal": "*@dev"
  }
}
```
Notice we use the `path` repository and we use `*@dev` in the modules we want to code LIVE!

Now we simply run `composer update`

Composer will **symlink** those 3 modules.  Look in `./vendor/acme` and you will see they are symlinked.  These modules are not ready to code LIVE! 
```
Package operations: 3 installs, 0 updates, 0 removals
  - Installing acme/wiki (dev-master): Symlinking from ./modules/acme/wiki
  - Installing acme/blog (dev-master): Symlinking from ./modules/acme/blog
  - Installing acme/dealer-portal (dev-master): Symlinking from ./modules/acme/dealer-portal
```

To "enable" these modules in mRcore, edit `config/modules.php` in the `modules` array like so
```php
'modules' => [
  ...
  'Acme\Wiki' => [],
  'Acme\Blog' => [],
  'Acme\DealerPortal' => [],
  ...
```

Now run `./artisan` and you will see extra acme commands.  This shows you the modules were loaded and the Console commands were registered with their providers
```
...
 acme
  acme:blog:app                  Mrcore app/module helper command
  acme:dealer-portal:app         Mrcore app/module helper command
  acme:wiki:app                  Mrcore app/module helper command
...
```

If these are to be Web UI modules, then all assets, views and routes must be loaded in the desired order.  Edit `config/modules.php` again
this time altering the "Loading Order / Override Management" section.  Add all 3 apps to the `assets`, `views`, and `routes` arrays, example:
```php
'assets' => [
  '%app%',
  'Acme\Wiki',
  'Acme\Blog',
  'Acme\DealerPortal',
]
```

You will also need to install a theme and probably auth
```
composer require mrcore/bootswatch-theme:5.7.*
composer require mrcore/auth:5.7.*
```

Now you can visit the various route prefixes for each module, example http://localhost:8000/wiki

For web apps, any public assets (css, html, images...) that you have in your module will be piped through the mRcore asset manager! This eliminates
the need to symlink into `./public` or `artisan publish` on every change.  For example http://localhost:8000/assets/app/acme/blog/css/home.css will be streamed via the asset manager!

**Have fun building all your web apps, API's and CLI's as reusable mRcore modules!**

----
----
----

## Versions

* 1.0 is for Laravel 5.1 and below
* 2.0 is for Laravel 5.3, 5.4, 5.5
* 5.6 is for Laravel 5.6
* 5.7 is for Laravel 5.7
* ... Following Laravel versions from here on

## Contributing

Thank you for considering contributing to the mRcore framework!  Fork and pull!

### License

mRcore is open source software licensed under the [MIT license](http://mreschke.com/license/mit)















