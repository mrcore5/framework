## Mrcore Framework v2.0

Mrcore is a set of Laravel components used to build various systems.
It is a framework, a development platform and a CMS.  It is a modularized version of Laravel
providing better package development support.  Think of Laravel 4.x workbenches on steroids.

Mrcore is simply Laravel + mrcore5/foundation which is an asset manager and dynamic module/app
loading system for laravel.

Once we have a proper asset and module system, plugging in modules to build the system or CMS or your
choice is extremely simple.

## Mrcore Available Modules

* https://github.com/mrcore5/foundation
* https://github.com/mrcore5/auth
* https://github.com/mrcore5/wiki




## Official Documentation

### Installation Foundation

Notice this git repo is empty!  That's because mrcore is simply Laravel + any number
of mrcore modules and apps you choose.  So you start with Laravel, and build your system manually.

* Assume you are installing to `/var/www/larabuild1`
* Create our directory structure `mkdir -p /var/www/larabuild1/{Apps,Files,Modules,Themes}`
 * This structure allows you to code apps, themes and modules as packages outside of composer and the vendor directory.  If you will not be coding, but instead simply use composer packages, you can skip these directories, even skip the System folder below and install all in the root like any Laravel project.
* Install Laravel 5.3 `cd /var/www/larabuild1 && composer create-project laravel/laravel System "5.3.*"`
 * At this point you should have a fresh working Laravel!  Setup your own apache2 or nginx site and test it out in your browser!  Your webserver should point to `/var/www/larabuild1/System/public`.
* Working from `cd /var/www/larabuild1/System`
* `chmod a+x ./artisan`
* Install mrcore foundation `composer require mrcore/foundation:~2.0`
* Manually edit your `config/app.php` file and add `Mrcore\Foundation\Providers\FoundationServiceProvider::class,` to your service providers (at the bottom of the 'providers' array)
 * While in `config/app.php` go ahead and remove or comment out all the those `App\Providers\*` Laravel lines (App, Auth, Event, Route).  This is optional, but they really are not needed.  We will never touch the actual Laravel app folder again.  All code is done in mrcore apps or modules now!
* Run the foundation installer script `./artisan mrcore:foundation:install`
 * This will include foundation bootstrap methods from within `bootstrap/autoload.php` which contains the asset manager and other helpers.
 * Publish the foundations `config/module.php` for your modification pleasure
 * Remove Laravel routes, views, models and migrations.  We won't be needing them.  **Careful here if this is not a stock laravel.**

Check your browser again.  Should see mRcore Foundation!


### Build Your First Web App

After installing the foundation above, assuming `/var/www/larabuild1` directory, you can create your first mrcore web app with the following.

* Install the bootswatch theme with `composer require mrcore/bootswatch-theme:~2.0` from the `System` directory
* Set `'enabled' => true,` for `BaseTheme` in `config/modules.php`
* Create the app called `Mreschke\Test` by running `./artisan mrcore:foundation:app:make mreschke/test`
* Edit `config/module.php` and add
```
'%app%' => [
	'type' => 'module',
	'namespace' => 'Mreschke\Test',
	'controller_namespace' => 'Mreschke\Test\Http\Controllers',
	'provider' => 'Mreschke\Test\Providers\TestServiceProvider',
	'path' => ['vendor/mreschke/test', '../Apps/Mreschke/Test'],
	'routes' => 'Http/routes.php',
	'route_prefix' => 'test',
	'views' => 'Views',
	'view_prefix' => 'test',
	'assets' => 'Assets',
	'enabled' => true,
],
```
* Now visit `/test` in our browser.
* Start coding in the `App/Mreschke/Test` folder.
* Mrcore apps come with a `cli`.  Just edit your `Mreschke/Test/Providers/TestServiceProvider` and uncomment the `registerCommand()` `AppCommand`.  Then you can run `./artisan` and see `mreschke:test:app` is now available.  Run it to see the help output.



### Install a wiki

After installing the foundation above, assuming `/var/www/larabuild1` directory, you can setup your own wiki using `mrcore/wiki` package.

* Install the wiki with `composer require mrcore/wiki:~2.0` from the `System` directory
* The wiki is seeded with 10 default posts, so `mkdir -p /var/www/larabuild1/Files/index/{1..10}`
* Manually edit your `config/app.php` and set timezone to `America/Chicago` or whatever your timezone is
* Manually edit your `config/auth.php` and set the guards web `'driver' => 'mrcore'` and the providers users `'model' => Mrcore\Auth\Models\User::class`
* Edit the `.env` to your liking
 * Add a `MRCORE_WIKI_WEBDAV_URL=webdav.example.com` key
 * Define the database, we'll call this one `mrcore5` on localhost
 * Set your mail settings properly and any other settings in `.end` or the `config/*` files
* Manually edit `config/modules.php` and set `'enable' => true'` for the `Auth, Wiki, Parser and BaseTheme` modules
* Migrate auth `./artisan mrcore:auth:app db:migrate`
* Migrate wiki `./artisan mrcore:wiki:app db:migrate`
* Seed auth `./artisan mrcore:auth:app db:seed`
* Seed wiki `./artisan mrcore:wiki:app db:seed`

**Success.** Check your browser.  Default login is `admin / password`.


#### Cron and Queue Worker

If this is a new system, you'll generally want these cron additions as your normal user (not root).

Run `crontab -e` as normal user and enter these lines

	# m h  dom mon dow   command
	  * *  *   *   *	 php /var/www/mrcore5/System/artisan schedule:run > /dev/null 2>&1
	  * *  *   *   *	 php /var/www/mrcore5/System/artisan queue:restart > /dev/null 2>&1
	  0 0  *   *   *	 /usr/local/bin/composer self-update > /dev/null
	  @hourly			php /var/www/mrcore5/System/artisan mrcore:wiki:index > /dev/null

Notice `mrcore5/wiki` has an hourly post indexer

#### Queue Worker

Some mrcore5 modules or some of your own personal apps will probably utilize Laravel queue workers.
I always run Ubuntu/Debian and utilize `systemd` or `supervisor` to handle my queues.


##### Systemd
This is more robust than supervisor and the prefered method.

Create a `/etc/systemd/system/mrcore5.service` systemd unit file

	# mrcore5 queue worker using systemd
	# ----------------------------------
	# /etc/systemd/system/mrcore5.service
	# systemctl enable mrcore5
	# systemctl start mrcore5

	[Unit]
	Description=mrcore5 queue worker

	[Service]
	User=toor
	Group=toor
	Restart=always
	RestartSec=3
	ExecStart=/usr/bin/php /var/www/mrcore5/System/artisan queue:work --daemon --sleep=3 --tries=3 --memory=1024

	[Install]
	WantedBy=multi-user.target

* Enable at start with `sudo systemctl enable mrcore5`
* Start with `sudo systemctl start mrcore5`.  Similar for `restart` and `stop`.
* If you edit the unit file, reload with `systemctl daemon-reload`



##### Supervisor
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




## Versions

* 1.0 is for Laravel 5.1
* 2.0 is for Laravel 5.3

## Contributing

Thank you for considering contributing to the mRcore framework!  Fork and pull!

### License

Mrcore is open-sourced software licensed under the [MIT license](http://mreschke.com/license/mit)
