# PHP Console server library

[![Author](http://img.shields.io/badge/author-@barbushin-blue.svg?style=flat-square)](https://www.linkedin.com/in/barbushin)
[![GitHub release](https://img.shields.io/github/release/barbushin/php-console.svg?maxAge=2592000&style=flat-square)](https://packagist.org/packages/php-console/php-console)
[![Coverage Status](https://img.shields.io/scrutinizer/coverage/g/barbushin/php-console.svg?style=flat-square)](https://scrutinizer-ci.com/g/barbushin/php-console/code-structure)
[![Build Status](https://img.shields.io/travis/barbushin/php-console/master.svg?style=flat-square)](https://travis-ci.org/barbushin/php-console)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)
[![Packagist](https://img.shields.io/packagist/dt/php-console/php-console.svg?maxAge=86400&style=flat-square)](https://packagist.org/packages/php-console/php-console)

PHP Console allows you to handle PHP errors & exceptions, dump variables, execute PHP code remotely and many other things using [Google Chrome extension PHP Console](https://chrome.google.com/webstore/detail/php-console/nfhmhhlpfleoednkpnnnkolmclajemef) and [PhpConsole server library](https://github.com/barbushin/php-console).

### Overview

* See [presentation video](http://www.youtube.com/watch?v=_4kG-Zrs2Io).
* Install Google Chrome extension [PHP Console](https://chrome.google.com/webstore/detail/php-console/nfhmhhlpfleoednkpnnnkolmclajemef).
* See how it works on [live demo](http://consle.com/instance/examples) page.
* PHP Console extension [features list and screenshots](https://github.com/barbushin/php-console/wiki/PHP-Console-extension-features).
* PHP Console server library [features list](https://github.com/barbushin/php-console/wiki/PHP-Console-server-features).
* Chrome extension source code https://github.com/barbushin/php-console-extension 

### Requirements

* [PHP Console extension](https://chrome.google.com/webstore/detail/php-console/nfhmhhlpfleoednkpnnnkolmclajemef) must be installed on Google Chrome ([extension source code](https://github.com/barbushin/php-console-extension) is also on GitHub)
* PHP 5.3 (or later) on server.

*For projects with PHP < 5.3 you can try to use old [deprecated version](https://groups.google.com/forum/?hl=ru#!forum/php-console-deprecated-version) of PHP Console. But mention that actual last version is much more functional.*

# Installation

### Composer

	{
		"require": {
			"adeotek/php-console": "^3.2"
		}
	}

Or

	$ composer require adeotek/php-console

### Monolog handler
https://github.com/Seldaek/monolog/blob/master/src/Monolog/Handler/PHPConsoleHandler.php

### Symfony framework bundle
https://github.com/Vitre/php-console-bundle

### Yii framework extension
http://www.yiiframework.com/extension/php-console

### Slim framework
https://github.com/amenadiel/SlimPHPConsole (thanks to [@amenadiel](https://github.com/amenadiel))

### Silex framework service provider
https://github.com/barbushin/php-console-silex (thanks to [@Chi-teck](https://github.com/Chi-teck))

### Laravel 4.* & 5.* framework service provider
https://github.com/barbushin/php-console-laravel

### Drupal CMS module
https://drupal.org/project/pc (thanks to [@Chi-teck](https://github.com/Chi-teck))

### WordPress plugin
https://github.com/unfulvio/wp-php-console (thanks to [@nekojira](https://github.com/unfulvio))

# Usage

You can try most of PHP Console features on [live demo](http://consle.com/instance/examples) server.

## Connector

There is a [PhpConsole\Connector](src/PhpConsole/Connector.php) class that initializes connection between PHP server and Google Chrome extension. Connection is initialized when [PhpConsole\Connector](src/PhpConsole/Connector.php) instance is initialized:

	$connector = PhpConsole\Connector::getInstance();

Also it will be initialized when you call `PhpConsole\Handler::getInstance()` or `PhpConsole\Helper::register()`.

### Communication protocol

PHP Console uses headers to communicate with client, so `PhpConsole\Connector::getInstance()` or `PhpConsole\Handler::getInstance()` must be called before any output. If headers are sent before script shut down or PHP Console response package size is out of web-server headers size limit, then PHP Console will store response data in [PhpConsole\Storage](src/PhpConsole/Storage.php) implementation and send it to client in STDOUT, in additional HTTP request. So there is no limit in PHP Console response package size.

### Troubleshooting with $_SESSION handler overridden in some frameworks

By default PHP Console uses [PhpConsole\Storage\Session](src/PhpConsole/Storage/Session.php) for postponed responses, so all temporary data will be stored in `$_SESSION`. But there is some problem with frameworks like [Symfony](http://symfony.com) and [Laravel](http://laravel.com) that overrides PHP session handler. In this case you should use any other [PhpConsole\Storage](src/PhpConsole/Storage.php) implementation like:

	// Can be called only before PhpConsole\Connector::getInstance() and PhpConsole\Handler::getInstance()
	PhpConsole\Connector::setPostponeStorage(new PhpConsole\Storage\File('/tmp/pc.data'));

See all available [PhpConsole\Storage](src/PhpConsole/Storage.php) implementations in [/src/PhpConsole/Storage](src/PhpConsole/Storage).

### Strip sources base path

If you want to see errors sources and traces paths more short, call:

	$connector->setSourcesBasePath('/path/to/project');

So paths like `/path/to/project/module/file.php` will be displayed on client as `/module/file.php`.

### Works with different server encodings

If your internal server encoding is not UTF-8, so you need to call:

	$connector->setServerEncoding('CP1251');

### Initialization performance

PhpConsole server library is optimized for lazy initialization only for clients that have Google Chrome extension PHP Console installed. There is [example](examples/features/highload_optimization.php) of correct initialization PhpConsole on your production server.

## Protect connection

### Protect by password

[![ScreenShot](http://consle.com/res/screenshot/auth_420.png)](http://consle.com/instance/examples/#protect_by_password)

	$connector->setPassword('yohoho123', true);

Clients will need to enter a password to get access to PHP Console server data. All passwords are stored on client as SHA-256 hashes. Second argument says that the PHP Console authorization token will depend on the client IP.

### SSL only connection mode

	$connector->enableSslOnlyMode();

So all PHP Console clients will be automatically redirected to HTTPS.

### Protect connection by list of allowed IP masks

	$connector->setAllowedIpMasks(array('192.168.*.*', '2001:0:5ef5:79fb:*:*:*:*'));

## Handle errors

[![ScreenShot](http://consle.com/res/screenshot/errors_420.png)](http://consle.com/instance/examples/#handle_errors)

There is a [PhpConsole\Handler](src/PhpConsole/Handler.php) class that initializes PHP errors & exceptions handlers and provides the next features:

* Handle PHP errors (+fatal & memory limit errors) and exceptions.
* Ignore repeated errors.
* Call previously defined errors and exceptions handlers.
* Handle caught exceptions using `$handler->handleException($exception)`.
* Debug vars using `$handler->debug($var, 'some.tags')`.

Initialize `PhpConsole\Handler` in the top of your main project script:

	$handler = PhpConsole\Handler::getInstance();
	/* You can override default Handler behavior:
		$handler->setHandleErrors(false);  // disable errors handling
		$handler->setHandleExceptions(false); // disable exceptions handling
		$handler->setCallOldHandlers(false); // disable passing errors & exceptions to prviously defined handlers
	*/
	$handler->start(); // initialize handlers


## Debug vars

[![ScreenShot](http://consle.com/res/screenshot/debug_420.png)](http://consle.com/instance/examples/#debug_vars)

PHP Console has multifunctional and smart vars dumper that allows to

* Dump any type variable.
* Dump protected and private objects properties.
* Limit dump by level, items count, item size and total size(see `$connector->getDumper()`).
* Dump objects class name.
* Smart dump of callbacks and Closure.
* Detect dump call source & trace(call `$connector->getDebugDispatcher()->detectTraceAndSource = true`).


### How to call

**Longest** native debug method call:

	PhpConsole\Connector::getInstance()->getDebugDispatcher()->dispatchDebug($var, 'some.tags');

**Shorter** call debug from Handler:

	PhpConsole\Handler::getInstance()->debug($var, 'some.tags');

**Shortest** call debug using global `PC` class

	PhpConsole\Helper::register(); // it will register global PC class
	// ...
	PC::debug($var, 'tag');
	PC::tag($var);

**Custom** call debug by user defined function

	function d($var, $tags = null) {
		PhpConsole\Connector::getInstance()->getDebugDispatcher()->dispatchDebug($var, $tags, 1);
	}
	d($var, 'some.tags');

### Tags

* Debug tags argument is optional.
* Tags is a string with tags separated by dot(e.g. "low.db").
* Tags can be used to identify what exactly var was dumped.
* You can configure client to ignore displaying some tags.

## Remote PHP code execution

[![ScreenShot](http://consle.com/res/screenshot/eval_terminal_420.png)](http://consle.com/instance/examples/#eval_terminal)

PHP Console provide a way to execute PHP code on your server remotely, from Google Chrome extension terminal.

* Remote PHP code execution allowed only in password protected mode
* Every eval request is signed with unique SHA-256 token
* Result contains: `output`, `return` and `time` data
* Errors & exception occurred during PHP code execution will be handled


### Configuration

	$connector = PhpConsole\Connector::getInstance();
	$connector->setPassword($password);

	// Configure eval provider
	$evalProvider = $connector->getEvalDispatcher()->getEvalProvider();
	$evalProvider->addSharedVar('post', $_POST); // so "return $post" code will return $_POST
	$evalProvider->setOpenBaseDirs(array(__DIR__)); // see http://php.net/open-basedir

	$connector->startEvalRequestsListener(); // must be called in the end of all configurations


## PSR-3 logger implementation

There is PHP Console implementation of [PSR-3](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md) interface. to integrate PHP Console with PSR-3 compitable loggers(e.g. [Monolog](https://github.com/Seldaek/monolog)). See [PhpConsole\PsrLogger](src/PhpConsole/PsrLogger.php).

## Jump to file

Read [this article](https://github.com/barbushin/php-console/wiki/Jump-to-file) if you want to configure PHP Console extension to open errors/exceptions source file:line right in your IDE, just by click on the button in Notification popup.

## Easy migrate from PhpConsole `v1.x` to `v3.x`

If you have used PhpConsole `v1.x` and want to migrate to `v3.x`  without any code changes, so just use [PhpConsole\OldVersionAdapter](src/PhpConsole/OldVersionAdapter.php):

	PhpConsole\OldVersionAdapter::register(); // register PhpConsole v1.x class emulator

	// Call old PhpConsole v1 methods as is
	PhpConsole::start(true, true, $_SERVER['DOCUMENT_ROOT']);
	PhpConsole::debug('Debug using old method PhpConsole::debug()', 'some,tags');
	debug('Debug using old function debug()', 'some,tags');
	echo $undefinedVar;
	PhpConsole::getInstance()->handleException(new Exception('test'));

	// Call new PhpConsole methods, if you want :)
	PhpConsole\Connector::getInstance()->setServerEncoding('cp1251');
	PhpConsole\Helper::register();
	PC::debug('Debug using new methods');

But, anyway, if you can't migrate to new version of PHP Console because of using PHP < 5.3 on your servers, then you can use old [deprecated version](https://groups.google.com/forum/?hl=ru#!forum/php-console-deprecated-version) of PHP Console.
