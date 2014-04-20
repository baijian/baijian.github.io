---
layout: post
title: "php-project-with-composer"
date: 2014-03-29 15:43:50
comments: true
categories: 
---

All java developers like to use `maven` to manage their project, it is very convenient, and many phpers will
ask if there is a tool like `maven` can help we phpers to manage our projects. Ofcourse it has, it's called
`Composer`.

[Composer](http://getcomposer.org) is a dependency manager for PHP, it help you declare, manage and install
dependencies of PHP projects. It is writen in php language completely.

### Install and use it as a linux command

First of all, we should install composer, you can install it within your single project, but I like to install it globally.

```
$ curl -sS https://getcomposer.org/installer | php -- --install-dir=$HOME/Bin
```

```
$ mv $HOME/Bin/composer.phar $HOME/Bin/composer
```

As always I do, I like to place some frequently-used command tools in my $HOME/Bin
directory, ofcourse $HOME/Bin is in my `PATH` variable.Then I can use it by just
inputing `composer` command.

### Init PHP web project using composer

PHP as a easy to use and high development efficiency language, a lot of medium and
small-sized enterprises like to use it as their main languages. With some caching
technology it will be a suitable solutions for them. You use php to support your
website you should select an open-source php framework, and you want to use
`composer`, your php framework must support `composer`.

I will recommend you a year's best php framework and use `composer` to
create and manage your web project based on it.
The framework is [Laravel](http://laravel.com/),it is not faster
than [yaf](http://www.yafdev.com/), but I think it is a very good php framework and
its efficiency is totally acceptable to us.

```
$ composer create-project laravel/laravel [ProjectName] --prefer-dist
```

If the command above execute for a long time, maybe you have another choice.Download [Laravel-PHAR-archive](http://laravel.com/laravel.phar)
and put it in your $HOME/Bin directory, then execute command below.

```
$ laravel new [ProjectName]
```

After execute the command above, the first level of your project directory is as below:

```
--app/
  --config // all configs, Config::get('file.arrkey', 'default'), different env with different sub-directory
  --models
  --views
  --controllers
  --start // some app startup init config, different env with env name php file
  filters.php // App or Route filters
  routes.php // Restfull router
  ...
--bootstrap/
  autoload.php // select vendor/autoload.php as autoload mechanism
  start.php // app startup script
  path.php
--public/
  index.php // entrance script
  ...
--vendor/ // place all dependencies of your project
artisan
composer.json
composer.lock
CONTRIBUTING.md
phpunit.xml
readme.md
server.php
```

run `php artison serve` to view website.

### Use composer in common PHP library project

I want to write a php library, first I will need to init my project with a file
called `composer.json` which is used to declare the library, below is my repo
[AhoCorasick-PHP's](https://github.com/baijian/AhoCorasick-PHP) `composer.json`.
With using `psr-4`, you can use namespace `namespace Baijian\Algorithm` without
making directory `Baijian` and `Alogorithm`, it's cool~

{% highlight json %}
{
    "name": "baijian/ahocorasick",
        "description": "",
        "keywords": ["ahocorasick"],
        "license": "",
        "authors": [
            {
                "name": "baijian",
                "email": "jian.baij@gmail.com",
                "role": "Learner"
            }
        ],
        "homepage": "http://joinjoy.me",
        "autoload": {
            "psr-4": {
                "Baijian\\Algorithm\\" : "src/"
            }
        },
        "require": {
            "php": ">=5.3.0"
        },
        "require-dev": {
            "phpunit/phpunit": "3.7.*"
        },
        "config": {
            "preferred-install": "dist"
        },
        "scripts": {
            "post-install-cmd": [
                "php index.php"  // execute after composer install
            ]
        }
}
{% endhighlight %}

Then execute `composer install --prefer-dist`, it will install the dependences in the folder of
`vendor`, and then create a file `composer.lock` for you.`vendor` should be ignored.
Then you created `src` folder and then write your code file in your `src` folder.At last in the
root directory of you project, you create a `index.php` file and require the `vendor/autoload.php`
file, then you can use the code in `src` folder to test your php library.

### Composer usage in common

All the commands of `composer` are [here](https://getcomposer.org/doc/03-cli.md).
And I will list some common commands.

* `composer selfupdate` is to update composer.

* `composer install --profile` the --profile is to see execute time of the execution.

* `composer update foo/bar` to update specific vendor for you.

* `composer update --lock` to update composer.lock file only.

* `composer require "foo/bar:0.0.1"` to add a vendor.

* `composer create-project vendor/project dir [version] --prefer-dist` to create a project, and setup from a dist package. Such as:`composer create-project laravel/laravel webdemo --prefer-dist` and dist packages are cached in $HOME/.composer directory automatically.

* `composer status -v` to see if you modify the code of your vendor direcotory, maybe sometimes you want to fix bug of one library, you can reserve it when `composer update`

* `composer dump-autoload --optimize` to optimize the autoload files before publish it.

At last you'd better know `scripts` config in `composer.json`, it can help you install some hooks(or callback) in some specific period.Such as `post-install-cmd` will be executed after you run `composer install`. All the period see [here](https://getcomposer.org/doc/articles/scripts.md)

### PHP learning resources

[PHP-The-Right-Way](http://www.phptherightway.com/)

[Know-your-http](https://github.com/bigcompany/know-your-http)

[PHP-standards](https://github.com/php-fig/fig-standards)

[Phpbrew-manage-php-versions](https://github.com/c9s/phpbrew)

[Zend-Framework](https://github.com/zendframework/zf2)

[Laravel-PHP-Framework](http://laravel.com/)

[Yaf-PHP-Framework](http://www.yafdev.com/)

[PHP-extension-dev-resources](http://www.laruence.com/2011/09/13/2139.html)

[PHP-extension-dev-guide](http://www.laruence.com/2009/04/28/719.html)

And some other good resources tell me ^-^.
