PHP Debug Mode Enabler
======================

Safe and clean way to manage Debug Mode in your app by specific environment and/or manually in App.
Package automatically detects development environments and provide secure way to temporary switch Debug Mode of your App
at any environment.

## Features
Package allows your app to switch to Debug Mode: 
- automatically on localhost's environment by IP,
- semi-automatically on any environment where you set `APP_DEBUG` environment variable (useful for Docker dev-stack), 
- semi-automatically disable Debug mode on any environment where you set `app-debug-mode` cookie variable (useful for
tests and similar cases), 
- manually enable/disable (force turn-on or turn-off) Debug Mode.

**NOTE:** Package is NOT provide any Debug tools directly – it only gives the app whether to switch to debug mode.

Package is optimized for invoking in very early lifecycle phase of your App and   

## Requirements
Package is requires:

- PHP version at least 7.3
- Temporary directory with writable access 

## Installation
```shell
composer require redbitcz/debug-mode-enabler
```

## Using
Anywhere in your app you can determine if app is running in Debug mode by simple code:
```php
$debugMode = \Redbitcz\DebugMode\Detector::detect($tempDir); //bool
```
where `$tempDir` is required path to temporary directory.

It returns `$debugMode` = `true` when is detected Debug environment or manually switched.

### Using with Nette
In Boostrap use package like this example:
```php
$tempDir = __DIR__ . '/../temp';
$debugModeDetector = new \Redbitcz\DebugMode\Detector($tempDir);

$configurator = new Configurator();
$configurator->setDebugMode($debugModeDetector->isDebugMode());
```
> I know, you love DI Container to build services like this. But Container Loader need to know Debug Mode state before is
> DI Container ready, you cannot use DI for Debug Mode detecting. 

## Using with Docker
If you building custom Docker image for your devstack, add the environment variable `APP_DEBUG=1`. For example in `Dockerfile` file:
```
ENV APP_DEBUG 1
```
> Avoid to publish these image to production!

## Using with Docker compose
In your devstack set environment variable `APP_DEBUG=1`. For example in `docker-compose.yml` file:
```yaml
environment:
    APP_DEBUG: 1
```

## Manually switch
**WARNING – DANGER ZONE:** Following feature allows to force Debug Mode on any environment, production including.
Please use it with great caution only! Wrong use might cause to critical security issue! Before using Enabler's feature be
aware your app is resistant to XSS, CSRF and similar attacks!  

Enabler provide feature to force enable or disable Debug Mode anywhere for user's browser (drived by Cookie). 

This example turn on Debug Mode for user's browser:
```php
$enabler = new \Redbitcz\DebugMode\Enabler($tempDir);
$enabler->activate(true);
```

### Options
- `$enabler->activate(true)` - force to Debug Mode turn on,
- `$enabler->activate(false)` - force to Debug Mode turn off,
- `$enabler->deactivate()` - reset back to automatically detection by environment.

### Using with Nette
Debug Mode Enabler (unlike Debug Mode Detector) can be simply served through DI Container with configuration in `config.neon`:
```yaml
services:
    - Redbitcz\DebugMode\Enabler(%tempDir%)
```

At most cases this example is creates second instance of `Enabler` class because first one is already created
internally with `Detector` instance in `Bootstrap`.

To re-use already exists instance you can inject it to DI Container:
```php
$tempDir = __DIR__ . '/../temp';
$debugModeDetector = new \Redbitcz\DebugMode\Detector($tempDir);

$configurator = new Configurator();
$configurator->setDebugMode($debugModeDetector->isDebugMode());
$configurator->addServices(['debugModeEnabler' => $debugModeDetector->getEnabler()]);
```

Don't forget let it know to DI Container with service declaration in `config.neon`: 
```yaml
services:
    debugModeEnabler:
        type: Redbitcz\DebugMode\Enabler
        imported: true
```  

## License
The MIT License (MIT). Please see [License File](LICENSE) for more information.