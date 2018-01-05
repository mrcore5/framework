## Mrcore Framework v2.0

Mrcore is basically a **module system for laravel** that lets you build **"packages"** easier
than managing providers in `config/app.php`,  messing with `public/` symlinks, constantly
publishing assets or trying to override views, assets and routes in different order
for different packages.

I think absolutely everything should be built as a package, even your "main" code.  Nothing should ever be coded
inside the main Laravel folder.  No `app/` code, no `migrations/`, no `resources/`, NOTHING.  Laravel
should simply be a shell (or a server) for packages.  This makes your code ultimately portable for re-use
inside other applications and lets you think "API first".

What if you wanted to build a Wiki.  Your wiki code should not just contain a GUI, it should also be a repository to
the wiki database...or an **API**.  What if you built another app#2, and that application needed to interact
with the wiki data.  Would you just query the wiki database directly from app#2?  No.  Instead you
`composer require wiki` into app#2 and use `$wiki->post->get()` repository style.  Can't do that if
your wiki code is a full Laravel application.  **All code must be packages for API re-use.**

So if you believe that, then trying to acheive this inside laravel can be difficult and frustrating because
of a few simple things.

One issue going it alone, is asset management.  If your code is in say `/packages/mrcore/wiki/Assets/images/hi.png` how
can you access that from the url?  With plain laravel, all you can do is `publish` (copy) those assets into Laravel's main
`public` folder.  This is obviously a pain and not real-time.  For a real-time hack you can just `symlink` your assets
into the `public` folder.  Symlinks are a hack and don't work on Windows and of course make your production deploy
more complicated.

The other issue, that you won't run into unless your app (think a CMS) has dozens of packages working together, is that
of override order.  By that I mean, if you have 12 packages, and each package has assets (css, js...), routes.php, views...
which one is used first?  Sometimes you want the main core CMS to have the last say for assets, but NOT for routes.  This
override problem can get complicated quickly.  With plain Laravel, all you can do is fire up your "packages" one at a time, in
order inside `config/app.php`.  This means assets, views, routes will all be "registered" together as a group, which can have
override order issues.

In order to solve all of these problems, you have to write your own custom package loading system + asset manager
to stream assets directly where they lay, no symlinking or copying.  This system must also be aware that packages can
reside in multiple places.  In local development, they may reside in a separate `../App/` folder but in production with
composer, they should reside in `vendor/`.

This is exactly why mrcore was built.

Documentation is lacking, but please install your first `laravel53` example below and dig into the `mrcore5/foundation`
code to see what is going on here.  The main bits in `mrcore5/foundation` are:
* What the installer does (inject the asset manager) https://github.com/mrcore5/foundation/blob/master/Console/Commands/InstallCommand.php
* The bootstrap which recognizes `/assets` https://github.com/mrcore5/foundation/blob/master/Bootstrap/Start.php
* The actual asset manager https://github.com/mrcore5/foundation/blob/master/Support/Assets.php
* The `config/modules.php` loading system https://github.com/mrcore5/foundation/blob/master/Support/Module.php
* Your actual module config in your `config/modules.php`



## Official Documentation

First example I show you how to start building your own application called `mreschke/app1`.
From there, you can build everything as a package and just wire them up in `config/module.php`

Second example I show you how to get a Wiki up and running.  This wiki is just another package, or "mrcore app"
thats included inside your Laravel install.

As a production example.  I use Laravel+mrcore at work for over 30 projects nearing 1 million lines of code.  One of my
projects is a "CMS" that is ONE Laravel install.  But that one install has over 20 "packages" or "mrcore apps" being used.
Some of these apps are ON all the time, like popular SSO API's, but most apps are only fired up if the URL matches
a certain URL, so they are lazy loaded modules based on URL.  We actually have hundreds of these little "wiki apps"
only fired up based on URL making them very efficient. This allow my CMD to not only contain pages and documents,
but also "apps" which are full folders of Laravel looking code. So `/wiki/about` might be a wiki page, but `/app/dashboard`
would be an entire 50k line application.  All are just modules inside one Laravel install.



### Folder Structure

I believe EVERYTHING should be a package.  I see Laravel as just a shell (or a server) to host and run your packages.
This means, for many projects, you can get away with ONE single Laravel install.  In these examples that one laravel install will be at
`/var/www/laravel53/System`.

Because of this, I like to relegate Laravel into it's own folder called `System` so I can focus more on my packages or `apps`.




This is the local DEV folder structure.  When you deploy to production, no need to have this structure.  For production,
you just install your packages into the regular `vendor` path as normal.

	/var/www/laravel53
		Apps
			Actual apps here example:
			Mreschke
				App1
				App2
			YourVendor
				App1
				App2
		Files
			App specific files here
		Modules
			Mrcore modules (like foundation) are here
		System
			Laravel is here
		Themes
			Your apps custom themes here

### Composer Autoloading

To actually code on your package, composer needs to be aware it and it needs to be autoloaded.  Think of it as if
your `Mreschke/App1` was actually in the `vendor/` folder as usual, so the main composer autoloader knows if it, but instead
its in the `Apps/Mreschke/App1` folder.  You can achieve this with composer's `path repository`.  With the path repository, if
composer finds the package in the defined path, it will actually symlink it into the vendor folder.  On occasiou, becuase of
some packagist website versioning interference this symlinking does not always work, so you can FORCE it in `pre-autoload-dump`.

Example `composer.json` file

```
{
	...
    "type": "project",
    "repositories": [
        {"type": "path", "url": "../Modules/*"},
        {"type": "path", "url": "../Themes/*"},
        {"type": "path", "url": "../Apps/*/*"}
    ],
    "require": {
        "php": ">=5.6.4",
        "laravel/framework": "5.3.*",

        "mreschke/app1": "*@dev",

    },
	...
    "scripts": {
		...
        "pre-autoload-dump": [
            "rm -rf vendor/mreschke/app1; ln -s ../../../Apps/Mreschke/App1 vendor/mreschke/app1",
        ]
    },
	...
}
```

Notice the new `path` repository, and your app defined as `*@dev` and the FORCE of symlink in case composers gets it wrong.  This allows
you to have just ONE `vendor` folder with one `autoloader` and have your projects code outside the `vendor` directory!  A perfect setup!


### Installation Foundation

Now that you have a good working folder structure with a SINGLE Laravel acting as a "package server" you can install `mrcore/foundation`.

* Assume you are installing to `/var/www/laravel53`
* Create our directory structure `mkdir -p /var/www/laravel53/{Apps,Files,Modules,Themes}`
 * This structure allows you to code apps, themes and modules as packages outside of composer and the vendor directory.  If you will not be coding, but instead simply use composer packages, you can skip these directories, even skip the System folder below and install all in the root like any Laravel project.
* Install Laravel 5.3 `cd /var/www/laravel53 && composer create-project laravel/laravel System "5.3.*"`
 * At this point you should have a fresh working Laravel!  Setup your own apache2 or nginx site and test it out in your browser!  Your webserver should point to `/var/www/laravel53/System/public`.
* Working from `cd /var/www/laravel53/System`
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

After installing the foundation above, assuming `/var/www/laravel53` directory, you can create your first mrcore web app with the following.

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

After installing the foundation above, assuming `/var/www/laravel53` directory, you can setup your own wiki using `mrcore/wiki` package.

* Install the wiki with `composer require mrcore/wiki:~2.0` from the `System` directory
* The wiki is seeded with 10 default posts, so `mkdir -p /var/www/laravel53/Files/index/{1..10}`
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
	# systemctl daemon-reload

	[Unit]
	Description=mrcore5 queue worker

	[Service]
	User=toor
	Group=toor
	Restart=always
	RestartSec=3
	ExecStart=/usr/bin/php /var/www/mrcore5/System/artisan queue:work --daemon --sleep=3 --tries=3 --timeout=600 --memory=1024

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
