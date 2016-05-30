## Mrcore5 Framework v1.x

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
of modules you choose.  So you start with Laravel, and build your system manually.

In this example, I will setup mrcore as a full wiki.  The wiki is amazing because not only does it
store your articles with fine grained permissions, but it also allows an article to be an entire
application, an entire Laravel application!  Think "full laravel package" but only registered and
fired up on defined routes.  This combination of wiki + apps = a simple and versatile CMS!

**For a live example and demo**, visit my personal website which is running mrcore wiki at http://mreschke.com

Lets build a wiki!

**Custom Directory Structure + Stock Laravel**

* Assume you are installing to `/var/www/mrcore5`
* Create our directory structure `mkdir -p /var/www/mrcore5/{Apps,Files,Modules,Themes}`
 * This structure allows you to code apps, themes and modules as packages outside of composer and the vendor directory.  If you will not be coding, but instead simply use composer packages, you can skip these directories, even skip the System folder below and install all in the root like any Laravel project.
* The wiki is seeded with 10 default posts, so `mkdir -p /var/www/mrcore5/Files/index/{1..10}`
* Install a fresh Laravel **into the System folder** `cd /var/www/mrcore5 && composer create-project laravel/laravel System "5.1.*"`

At this point you should have a fresh working Laravel!  Setup your own apache2 or nginx site and test it out in your browser!  Your webserver should point to `/var/www/mrcore5/System/public`.

**Add mrcore/wiki**

* Work from the System directory `cd /var/www/mrcore5/System`
* Add the wiki `composer require mrcore/wiki:~1.0` this will automatically pull in all required modules like foundation, auth, parser...
 * Ignore any composer Ambiguous warnings at this point
* Manually edit your `config/app.php` file and add `Mrcore\Foundation\Providers\FoundationServiceProvider::class,` to your service providers (at the bottom of the 'providers' array)
* While in `config/app.php` go ahead and remove or comment out all the those `App\Providers\*` laravel lines (App, Auth, Event, Route).  This is optional, but they really are not needed.  We will never touch the actual laravel app folder again.  All code is done in mrcore apps or modules now!
* Run `chmod a+x artisan`
* Run the foundation installer script `./artisan mrcore:foundation:install`
 * This will include foundation bootstrap methods from within `bootstrap/autoload.php` which contains the asset manager and other helpers.
 * Publish the foundations `config/module.php` for your modification pleasure
 * Remove Laravel routes, views, models and migrations.  We won't be needing them.  **Careful here if this is not a stock laravel.**
* Run `./artisan optimize`

Check your browser again.  Should see mRcore Foundation!
At this point you have the minimum base of mrcore.  From here your project can take on many forms
based on what you do next.  We'll be turning this install into a wiki!

**Configure Modules**

* Manually edit your `config/app.php` and add a env() to url like so `'url' => env('APP_URL', 'http://localhost')`
* Manually edit your `config/auth.php` and set the guards web `'driver' => 'mrcore'` and the providers users `'model' => Mrcore\Auth\Models\User::class`
* Edit the `.env` to your liking
	* Add a `APP_URL=http://example.com` key
	* Add a `MRCORE_WIKI_WEBDAV_URL=webdav.example.com` key
	* Define the database, we'll call this one `mrcore5` on localhost
* Manually edit `config/modules.php` and set `'enable' => true'` for the `Auth, Wiki, Parser and BaseTheme` modules
* Migrate auth and wiki `./artisan mrcore:auth:app db:migrate` and `./artisan mrcore:wiki:app db:migrate`
* Seed auth and wiki `./artisan mrcore:auth:app db:seed` and `./artisan mrcore:wiki:app db:seed`

**Success.** Check your browser.  Default login is `admin / password`.

#### Cron and Queue Worker

If this is a new system, you'll generally want these cron additions as your normal user (not root).

Run `crontab -e` as normal user and enter these lines

	# m h  dom mon dow   command
	  * *  *   *   *     php /var/www/mrcore5/System/artisan schedule:run > /dev/null 2>&1
	  * *  *   *   *     php /var/www/mrcore5/System/artisan queue:restart > /dev/null 2>&1
	  0 0  *   *   *     /usr/local/bin/composer self-update > /dev/null
	  @hourly            php /var/www/mrcore5/System/artisan mrcore:wiki:index > /dev/null

Notice `mrcore5/wiki` has an hourly post indexer

#### Queue Worker

Some mrcore5 modules or some of your own personal apps will probably utilize Laravel queue workers.
I always run Ubuntu/Debian and utilize `supervisor` to handle my queues.  

Create a `/etc/supervisor/conf.d/mrcore5_queue.conf` like so

	[program:mrcore5_queue]
	process_name=%(program_name)s_%(process_num)02d
	command=php /var/www/mrcore5/System/artisan queue:work --daemon --tries=3 --sleep=5 --memory=512
	autostart=true
	autorestart=true
	user=toor
	numprocs=1
	redirect_stderr=true
	stdout_logfile=/var/www/mrcore5/System/storage/logs/queue.log

And restart supervisor with `sudo service supervisor restart`.
You can run `ps aux | queue:work` to verify the worker is indeed running.

Because I use `--daemon` mode for my workers, I like to restart them every minute.  You'll notice
a `queue:restart` cron line above.






### Modules

**fixme - reword**

With the module system, you can build your apps or "packages" outside of the vendor folder.
Each app, like an old "workbench" has controllers, views, assets, events, migrations, seeds...
With the module loading system, these assets can override each other and the order can be defined
in a simple config.   Ever tried to simply register your "package" in `config/app.php` and find
views, routes and assets don't override the way you want, or in the proper order.  With the
modules system you can define the order of each modules assets, views and routes.  





### Performance

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
