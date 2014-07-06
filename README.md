# php-apkparser
A small library for parsing Android application package files (APKs).


Requirements
============
It is possible that these may or may not be necessary, or perhaps require a small modification to work.
* [PHP 5.3.0](http://php.net/releases/5_3_0.php)+ · the use of the 'const' keyword outside of classes


Usage
=====

Simple
------
To display some basic information about the package:

```php
<?php

require_once 'lib/APKParser/parser.php';

$apk = new \APKParser\APK('example.apk');

printf("-name = '%s'\n", $apk->get_package());
printf("-version = '%s (%s)'\n", $apk->get_androidversion_name(), $apk->get_androidversion_code());
printf("-min_sdk_version = '%s'\n", $apk->get_min_sdk_version());
```

Manifest
--------
You may also get the `AndroidManifest.xml` SimpleXML object:

```php
$apk = new \APKParser\APK('example.apk');
$manifest = $apk->get_android_manifest_xml();

printf("-object = '%s'", get_class($manifest));
```

...or perhaps you wish to get 'AndroidManifest.xml' as a string:

```php
$manifest = $apk->get_android_manifest_axml()->get_buff();
```

...OR YOU WANT AN ELEMENT ATTRIBUTE VALUE:

```php
$app_name = $apk->get_element('application', 'name', true);
printf("-application_name = '%s%s'", $apk->get_package(), $app_name);
```

Resources
---------
Some applications define their attribute values as references:
```php
printf("%s", $apk->get_androidversion_name()); // prints '@<hex>', e.g.: '@7f0b000d'
```

These references have to be decoded:

```php
$arscobj = $apk->get_android_resources();
$vn_res_id = substr($apk->get_androidversion_name(), 1);
printf("%s", $arscobj->get_resource_value_by_reference($vn_res_id));
```

References can point to files inside the package:

```php
$app_icon = $apk->get_element('application', 'icon', true); // @<hex>

$arscobj = $apk->get_android_resources();
var_export($arscobj->get_resource_value_by_reference(substr($app_icon, 1)));
/*
array (
  0 => 'res/drawable-mdpi/mushrooms.png',
  1 => 'res/drawable-hdpi/mushrooms.png',
  2 => 'res/drawable-xhdpi/mushrooms.png',
)
 */
```

...which you can get as a stream from the package (and then use as you wish):

```php
$icon = $arscobj->get_resource_value_by_reference(substr($app_icon, 1))[0];
$icon_stream = $apk->get_file($icon);
$base64_string = base64_encode(stream_get_contents($icon_stream));

printf("data:image/png;base64,%s", $base64_string);
```

Acknowledgements
================
* the [Androguard](https://code.google.com/p/androguard/) project from which parts of the code was ported from