# Purify

[![Travis CI](https://img.shields.io/travis/stevebauman/purify.svg?style=flat-square)](https://travis-ci.org/stevebauman/purify)
[![Scrutinizer Code Quality](https://img.shields.io/scrutinizer/g/stevebauman/purify.svg?style=flat-square)](https://scrutinizer-ci.com/g/stevebauman/purify/?branch=master)
[![Latest Stable Version](https://img.shields.io/packagist/v/stevebauman/purify.svg?style=flat-square)](https://packagist.org/packages/stevebauman/purify)
[![Total Downloads](https://img.shields.io/packagist/dt/stevebauman/purify.svg?style=flat-square)](https://packagist.org/packages/stevebauman/purify)
[![License](https://img.shields.io/packagist/l/stevebauman/purify.svg?style=flat-square)](https://packagist.org/packages/stevebauman/purify)

Purify is an HTML input sanitizer for Laravel.

It utilizes [HTMLPurifier](https://github.com/ezyang/htmlpurifier)
by [ezyang](https://github.com/ezyang).

### Installation

To install Purify, insert the following require in your `composer.json` file:

```json
"stevebauman/purify": "2.0.*"
```

Now run a `composer update` on your project source.

> **Note:** If you're using Laravel 5.5, ignore the below service provider and facade setup.
>
> These are registered automatically.

Then, insert the service provider in your `config/app.php`:

```php
'Stevebauman\Purify\PurifyServiceProvider'
```
    
You can also use the facade if you wish:

```php
'Purify' => 'Stevebauman\Purify\Facades\Purify'
```

Then, publish the configuration file using:

```cmd
php artisan vendor:publish --provider="Stevebauman\Purify\PurifyServiceProvider"
```

### Usage

##### Cleaning a String

To clean a users input, simply use the clean method:

```php
$input = '<script>alert("Harmful Script");</script> <p style="a style" class="a-different-class">Test</p>';

$cleaned = Purify::clean($input);

echo $cleaned; // Returns '<p class="a-different-class">Test</p>'
```

##### Cleaning an Array

Need to purify an array of user input? Just pass in an array:

```php
$array = [
    '<script>alert("Harmful Script");</script> <p style="a style" class="a-different-class">Test</p>',
    '<script>alert("Harmful Script");</script> <p style="a style" class="a-different-class">Test</p>',
];

$cleaned = Purify::clean($array);

var_dump($cleaned); // Returns [0] => '<p class="a-different-class">Test</p>' [1] => '<p class="a-different-class">Test</p>'
```

##### Dynamic Configuration

Need a different configuration for a single input? Pass in a configuration array into the second parameter:

```php
$config = ['HTML.Allowed' => 'div,b,a[href]'];

$cleaned = Purify::clean($input, $config);
```

> **Note**: Configuration passed into the second parameter is
> **not** merged with your current configuration.

```php
$config = ['HTML.Allowed' => 'div,b,a[href]'];

$cleaned = Purify::clean($input, $config);
```

##### Replacing the HTML Purifier instance

Need to replace the HTML Purifier instance with your own? Call the `setPurifier()` method:

```php
$purifier = new HTMLPurifier();

Purify::setPurifier($purifier);
```

### Configuration

Inside the configuration file, the entire settings array is passed directly
to the HTML Purifier configuration, so feel free to customize it however
you wish. For the configuration documentation, please visit the
HTML Purifier Website:

http://htmlpurifier.org/live/configdoc/plain.html

#### Custom Configuration Rules

There's mutliple ways of creating custom rules on the HTML Purifier instance.

Below is an example service provider you can use as a starting point to add rules to the instance. This provider gives compatibility with Basecamp's Trix WYSIWYG editor:

```php
<?php

namespace App\Providers;

use HTMLPurifier_HTMLDefinition;
use Stevebauman\Purify\Facades\Purify;
use Illuminate\Support\ServiceProvider;

class PurifySetupProvider extends ServiceProvider
{
    const DEFINITION_ID = 'trix-editor';
    const DEFINITION_REV = 1;

    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        /** @var \HTMLPurifier $purifier */
        $purifier = Purify::getPurifier();

        /** @var \HTMLPurifier_Config $config */
        $config = $purifier->config;

        $config->set('HTML.DefinitionID', static::DEFINITION_ID);
        $config->set('HTML.DefinitionRev', static::DEFINITION_REV);

        if ($def = $config->maybeGetRawHTMLDefinition()) {
            $this->setupDefinitions($def);
        }

        $purifier->config = $config;
    }

    /**
     * Register the application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Adds elements and attributes to the HTML purifier
     * definition required by the trix editor.
     *
     * @param HTMLPurifier_HTMLDefinition $def
     */
    protected function setupDefinitions(HTMLPurifier_HTMLDefinition $def)
    {
        $def->addElement('figure', 'Block', 'Flow', 'Common');
        $def->addElement('figcaption', 'Block', 'Flow', 'Common');
        $def->addElement('span', 'Block', 'Flow', 'Common');

        $def->addAttribute('figure', 'class', 'Text');
        $def->addAttribute('figcaption', 'class', 'Text');

        $def->addAttribute('a', 'rel', 'Text');
        $def->addAttribute('a', 'data-trix-attachment', 'Text');
    }
}
```

After this service provider is created, make sure you insert it into your `providers` array in the `app/config.php`
file, and update your `HTML.Allowed` string in the `config/purify.php` file.
