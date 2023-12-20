# A Code Encrypter for the Laravel Framework

The goal of this open source package is **security _through_ obscurity**.

It aims to offer an alternative to delivering your closed source projects in **plain text**. Instead, you can opt to deliver your files **encrypted**, alongside a binary PHP extension which will decrypt them on the fly.

This package uses symmetric encryption, which means that the key itself _has_ to be embedded within the binary PHP extension. However, the code for the extension which holds the key and handles the decryption is open source (and is included within this package).

The key is generated by you and only known to you (as the developer). It can be unique per project and/or customer which opens up the possibility of additional functionality later down the line, such as setting expiry dates (for free trials) and/or whitelisting by IP/MAC address.

## Requirements

1. macOS
2. PHP 8.1
3. [Laravel 10](https://laravel.com)
4. [Zephir](https://zephir-lang.com)

## Installation

The below assumes that you're currently in your application's root directory.

```console
$ composer require chrisvpearse/code-encrypter-laravel
```

### Publish the Config File

```console
$ php artisan vendor:publish --tag=code-encrypter-config
```

### Zephir Quickstart

You should be able to get started with Zephir by downloading the [latest release PHAR](https://github.com/zephir-lang/zephir/releases) from GitHub, followed by these commands:

```console
$ cd ~ && pecl install zephir_parser
```

```console
$ cd ~/Downloads && mv zephir.phar /usr/local/bin/zephir && chmod +x /usr/local/bin/zephir
```

## A "Hello, World!" Walkthrough

Again, the below assumes that you're currently in your application's root directory.

### Make an Invokable Controller

```console
$ php artisan make:controller HelloWorld --invokable
```

:page_facing_up: **./app/Http/Controllers/HelloWorld.php**

```php
<?php

namespace App\Http\Controllers;

class HelloWorld extends Controller
{
    public function __invoke()
    {
        return 'Hello, World!';
    }
}

```

### Update the Web Routes File

:page_facing_up: **./routes/web.php**

```php
<?php

use App\Http\Controllers\HelloWorld;
use Illuminate\Support\Facades\Route;

Route::get('/', HelloWorld::class);

```

### Update the Config File

The default configuration file is shown below.

* The `*` wildcard matches all files within the specified directory
* The `**` wildcard is recursive and matches all files within the specified directory and subdirectories

:page_facing_up: **./config/code-encrypter.php**

```php
<?php

return [
    'paths' => [
        app_path('Http/Controllers/Foo/*'),
        app_path('Http/Controllers/Bar/**'),
        app_path('Http/Controllers/HelloWorld.php'),
    ],
    'cipher' => config('app.cipher') ?: 'AES-256-CBC',
    'minify' => false,
];

```

A PHP file is considered valid provided that the following conditions are met:

* The extension is: **.php**
* It contains an opening PHP tag: `<?php`
* It does not contain a closing PHP tag: `?>`

### First Run

You should confirm that you can see "**Hello, World!**" after running the following command:

```console
$ php artisan serve
```

### Run the Encrypt Command

```console
$ php artisan code:encrypt
```

The output from the above command will be similar to the following:

```
Encrypted ....................................................... app/Http/Controllers/HelloWorld.php

[ INFO ] Command successfully completed.

Key ............................................. base64:8dsiWW4Ue016d1TBalTlJAE46vfO91y5/Xu9UFZzhmc=
Cipher .................................................................................. AES-256-CBC
Zephir File .......................................... tmp/code-encrypter/zephir/zephir/encrypter.zep
Zephir Temp Directory (Must Delete) .............................................. tmp/code-encrypter
```

:exclamation: Please remember to make a note of your **key**.

On this run, one file has been successfully encrypted:

:page_facing_up: **./app/Http/Controllers/HelloWorld.php**

```php
<?php \Zephir\Encrypter::decrypt("c3DHmSPBpjBUQcZ...8+SePRyCXaKbg==", "hj54ztR3KB+7cWKVJFP0QA==");
// base64:eyJpdiI6ImhqNTR6dFIzS0IrN2NXS1...UwZGVhYTY2MDRhIiwidGFnIjoiIn0=

```

The `decrypt()` static method in `\Zephir\Encrypter` takes the **value** and **iv** (initialization vector) from the payload returned by the `encryptString()` method in `\Illuminate\Encryption\Encrypter`.

The commented-out base64 encoded string contains the entire JSON payload returned by the `encryptString()` method in `\Illuminate\Encryption\Encrypter` and is used in the decryption process.

### A "Utils" Gist

For your convenience, I have published a [GitHub Gist](https://gist.github.com/chrisvpearse/e06b149b8055216dd954eb9b64750635) which exposes a small number of utilities which will be used throughout the rest of this document.

### Build the PHP Extension

At this point, you must build the PHP extension because Laravel will no longer be able to find `\App\Http\Controllers\HelloWorld` (which will result in an error).

You should switch to the **Zephir Temp Directory** as detailed on the output from the previous command:

```console
$ cd tmp/code-encrypter
```

From there, you should initialize Zephir _into_ the **zephir** directory which has already been created for you:

```console
$ zephir init zephir
```

Finally, you should step into the **zephir** directory and build the extension from there:

```console
$ cd zephir && zephir build
```

Once the extension has been built, you should be prompted to add `extension=zephir.so` to your **php.ini** file. For this, I will use the **ini** utility exposed by the GitHub Gist:

```console
$ php ~/Downloads/utils.php ini
```

Next, we can take a look at the Zephir code itself:

:page_facing_up: **./tmp/code-encrypter/zephir/zephir/encrypter.zep**

```
namespace Zephir;

class Encrypter
{
    public static function decrypt(var gLIEokrXlBB6s, var LxARS8923aokLv)
    {
        return eval(self::ssjgKzuSCrh9Zjhom1P(gLIEokrXlBB6s, LxARS8923aokLv));
    }

    ...

    protected static function ssjgKzuSCrh9Zjhom1P(var gLIEokrXlBB6s, var LxARS8923aokLv)
    {
        var ZURmZSyHXCWAwv;

        let ZURmZSyHXCWAwv = ["f1","db","22","59","6e","14",..."bd","50","56","73","86","67"];

        return openssl_decrypt(
            gLIEokrXlBB6s,
            "AES-256-CBC",
            hex2bin(implode(ZURmZSyHXCWAwv)),
            0,
            base64_decode(LxARS8923aokLv)
        );
    }

    ...
}

```

The above code is _not_ minified, which can (and should) be changed in **./config/code-encrypter.php**. During the minifying process, whitespace is randomly added to ensure that the key is never in the same place twice (within the binary PHP extension).

In an effort to further randomize the binary, the `\Illuminate\Support\Str::password()` helper is used to name the variables and methods. Additionally, the method which holds the key is shuffled with 4–8 unused methods (each containing their own key). Each key is stored within its own array to avoid being detected by the [strings](https://www.unix.com/man-page/osx/1/strings) command:

```console
$ strings /path/to/zephir.so
```

Now, let's use the **tmp** utility exposed by the GitHub Gist to delete the **Zephir Temp Directory**:

```console
$ php ~/Downloads/utils.php tmp "/path/to/app/root/"
```

### Second Run

You should confirm that you are still able to see "**Hello, World!**" after running the following command:

```console
$ cd ../../../ && php artisan serve
```

Congratulations :tada: Your encrypted files are now being decrypted on the fly!

### Run the Decrypt Command

In order to decrypt your code, you must supply the key as the first argument:

```console
$ php artisan code:decrypt base64:8dsiWW4Ue016d1TBalTlJAE46vfO91y5/Xu9UFZzhmc=
```

The output from the above command will be similar to the following:

```
Decrypted ....................................................... app/Http/Controllers/HelloWorld.php

[ INFO ] Command successfully completed.

```

On this run, one file has been successfully decrypted:

:page_facing_up: **./app/Http/Controllers/HelloWorld.php**

```php
<?php

namespace App\Http\Controllers;

class HelloWorld extends Controller
{
    public function __invoke()
    {
        return 'Hello, World!';
    }
}

```

### Run the Encrypt Command (Again)

Once the PHP extension has been built by Zephir, you can decrypt and encrypt your code as many times as you like without rebuilding the extension (provided that you use the same key). The key should be supplied via an option to the command:

```console
$ php artisan code:encrypt --key=base64:8dsiWW4Ue016d1TBalTlJAE46vfO91y5/Xu9UFZzhmc=
```

Note. When encrypting code, the **./tmp/code-encrypter** directory will always be generated for you.

## NativePHP

At the time of writing, NativePHP ships with a static PHP binary, therefore it is not possible to install additional extensions.

However, as a PoC, we can swap out the NativePHP binary with a copy of our local PHP binary (NativePHP must already be installed). For this, I will use the **bin** utility exposed by the GitHub Gist:

```console
$ php ~/Downloads/utils.php bin "/path/to/app/root/"
```

### Third Run

You should confirm that you are still able to see "**Hello, World!**" after running the following command:

```console
$ php artisan native:serve
```

Congratulations, again :confetti_ball: Your encrypted files are now being decrypted on the fly within **NativePHP**!

## Credits

* **Aleksandar Jevremović, et al.** (2013). _Using Cryptology Models for Protecting PHP Source Code._
