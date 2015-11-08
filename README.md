## Mrcore5 Framework

Mrcore is a set of Laravel and Lumen components used to build various systems.
It is a framework, a development platform and a CMS.  It is a modularized version of Laravel
providing better package development support.  Think of Laravel 4.x workbenches on steroids.

Mrcore is simply Laravel + mrcore5/foundation which is an asset manager and dynamic module/app
loading system for laravel.

Once we have a proper asset and module system, plugging in modules to build the system or CMS or your
choice is extremely simple.

## Mrcore5 Available Modules

* https://github.com/mrcore5/foundation
* https://github.com/mrcore5/auth
* https://github.com/mrcore5/wiki

## Official Documentation

### Installation

Notice this git repo is empty?  Thats becuase mrcore is simply Laravel + any number
of modules you choose.  So you start with laravel, and build your system manually.

In this example, I will setup mrcore as a full wiki.  The wiki is amazing because not only does it
store your article with fine grained permissions, but it also allows an article to be an entire
application, an entire Laravel application!  Think "workbenches" but only fired up on defined routes.
This combination of wiki + apps = a simple and versatile CMS!

Lets build a wiki!


**Custom Directory Structure + Stock Laravel**

* Assume you are installing to `/var/www/mrcore5`
* Create our directory structure `mkdir -p /var/www/mrcore5/{Apps,Files,Modules,Themes}`
* The wiki is seeded with 10 default posts, so `mkdir -p /var/www/mrcore5/Files/index/{1..10}`
* Install a fresh Laravel **into the System folder** `cd /var/www/mrcore5 && composer create-project laravel/laravel --prefer-dist System`

At this point, fresh laravel!  Setup your own apache2 or nginx site and test it out in your browser!


**Add mrcore5/foundation asset and module system**

* Add mrcore5/foundation `composer require mrcore/foundation:~1.0`
* Manually edit your `config/app.php` file and add `Mrcore\Modules\Foundation\Providers\FoundationServiceProvider::class,` to your service providers (at the bottom)
* While in `config/app.php` go ahead and remove or comment out all the those App\Provider\* laravel lines (App, Auth, Event, Route).  This is optional, but they really are not needed.  We will never touch the actual laravel app folder again.  All code is done in mrcore apps or modules!
* Run the foundation installer script `./artisan mrcore:foundation:install`
	* This will add the asset manager to `public/index.php`
	* Public foundation `config/module.php` for your modification pleasure
	* Remove laravels routes, views, models and migrations.  We won't be needing them.  Careful here if this is not a stock laravel.

Check your browser again.  Should see mRcore Foundation!
At this point you have the minimum base of mrcore.  From here your project can take on many forms
based on what youd do next.  We'll be turning this install into a wiki!

**Add auth, wiki and theme system**

* Add components `composer require mrcore/auth:~1.0 mrcore/wiki:~1.0 mrcore/bootswatch-theme:~1.0`
* Manually edit your `config/auth.php` and set `'driver' =>  'mrcore'` and `'model' => Mrcore\Modules\Wiki\Models\User::class`
	* Heres a little sed magic if you want
	* `sed -i "s/'driver'*/'driver' => 'mrcore'/" /var/www/mrcore5/System/config/auth.php`
	* `sed -i "s/'model'*/'model' => Mrcore\\\\Modules\\\\Wiki\\\\Models\\\\User::class/" /var/www/mrcore5/System/config/auth.php`
* Edit the `.env` to your liking
	* All the wiki needs is a database defined here.  Give it whatever name you want.  I call it `mrcore5`
	* I also use `redis` for all my `cache`, `session`, and `queue` drivers, but up to you
* Manually edit `config/modules.php` and set `'enable' => true'` for the Auth, Wiki, BaseTheme modules
* Run the mrcore5/wiki migrations `./artisan mrcore:wiki:db migrate`
* Run the mrcore5/wiki seeders `./artisan mrcore:wiki:db seed`

**Success.** Check your browser.  Default login is `admin / password`.


#### Modules

**fixme - reword**

With the module system, you can build your apps or "packages" outside of the vendor folder.
Each app, like an old "workbench" has controllers, views, assets, events, migrations, seeds...
With the module loading system, these assets can override each other and the order can be defined
in a simple config.   Ever tried to simply register your "package" in `config/app.php` and find
views, routes and assets don't override the way you want, or in the proper order.  With the
modules system you can define the order of each modules assets, views and routes.  


  
#### Performance

Tested with apache benchmark tool, a very simple `ab -n 10 -c 1`

* Stock laravel `17ms`
* Mrcore/Foundation `17ms`
* Mrcore/Wiki with complete working wiki home page `60ms`
  * Found that 10ms is taken by simply updating post and router clicks (view counts), would be `50ms`
  * At some point, may optimize these types of updates into a redis counter.



## Contributing

Thank you for considering contributing to the mRcore framework!  Fork and pull!

### License

Mrcore is open-sourced software licensed under the [MIT license](http://mreschke.com/license/mit)
