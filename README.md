## Mrcore5 Framework

Mrcore is a set of Laravel and Lumen components used to build various systems.
It is a framework, a development platform and a CMS.  It is my modularized version of Laravel!

## Mrcore5 Modules

Mrcore framework is simply Laravel + modules.

* https://github.com/mrcore5/foundation
* https://github.com/mrcore5/auth
* https://github.com/mrcore5/wiki

## Official Documentation

### Installation

* Pick base dir (ex: /var/www/blog).  Optional mkdir Apps, Files, Modules, Themes
* Fresh laravel 5.1 inside System folder
* Add foundation `composer require mrcore/foundation:~1.0`
* Add `Mrcore\Modules\Foundation\Providers\FoundationServiceProvider::class,` to config/app.php
* You can actually comment all the those App\Provider laravel lines (App, Auth, Event, Route)
* Install foundation `./artisan mrcore:foundation:install`
* Visit in browser, will say mRcore Foundation!
* Add components `composer require mrcore/auth:~1.0 mrcore/wiki:~1.0 mrcore/bootswatch-theme:~1.0`
* Set config/auth.php driver to mrcore and model to Mrcore\Modules\Wiki\Models\User::class
  * `sed -i "s/'driver'*/'driver' => 'mrcore'/" config/auth.php`
  * `sed -i "s/'model'*/'model' => Mrcore\\\\Modules\\\\Wiki\\\\Models\\\\User::class/" config/auth.php`
* Edit `.env` to your liking
* Edit `config/modules.php` and enable Auth, Wiki, BaseTheme
* Migrate wiki `./artisan mrcore:wiki:db migrate`
* Seed wiki `./artisan mrcore:wiki:db seed`
* Visit in browser, done!

  
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
