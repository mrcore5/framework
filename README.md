# Mrcore5 From Scratch

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




# Investigate

* In mrcore5 app, I comment out the Http/Kernel.php Csrf Token
  * Found, if you edit post, set workbench, get CSRF token error


  
# Performance

All just `ab -n 10 -c 1` on lindev1

* Stock laravel 17ms
* Mrcore/Foundation 17ms
* Full wiki 60ms
  * You can shave 10ms by NOT updateing clicks on both post and router = 50ms!
