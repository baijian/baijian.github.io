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
$ curl -sS https://getcomposer.org/installer | php -- --install-dir=$HOME/bin
```

```
$ mv $HOME/bin/composer.phar $HOME/bin/composer
```

As always I do, I like to place some frequently-used command tools in my $HOME/bin
directory, ofcourse $HOME/bin is in my `PATH` variable.Then I can use it by just
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

After execute the command above, the first level of your project directory is as below:

```
--app/
--bootstrap/
--public/
--vendor/
artisan
composer.json
composer.lock
CONTRIBUTING.md
phpunit.xml
readme.md
server.php
```

### Use composer in other common PHP project

### Composer usage in detail
