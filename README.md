# WP Intervention

WP Intervention provides on-demand image manipulation for WordPress via the [Intervention Library](http://image.intervention.io/).

## Why?

Traditionally, WordPress only provides fairly basic image manipulation facilities (eg: resize, crop). But what if you need to do more with your images? This is where WP Intervention comes in.

With WP Intervention you have the full suite of PHP Intervention's [powerful image manipulation tools](http://image.intervention.io/) at your finger tips including:

* [Blur](http://image.intervention.io/api/blur).
* [Grayscale](http://image.intervention.io/api/greyscale).
* [Mask](http://image.intervention.io/api/mask).

...and many more!

Resulting images are cached to avoid any further unnecessary processing.

## How?

All image manipulation is provided by the [PHP Intervention Library](http://image.intervention.io/).

WP Intervention merely provides a wrapper around Intervention and a convenient global function for use with WordPress.

This Plugin is currently aimed at __developers__ and aims to be fully customisable via liberal usage of [WordPress filters](https://developer.wordpress.org/reference/functions/apply_filters/).


## Installation

1. [Download a copy of this
repository](https://github.com/getdave/wp-intervention/archive/master.zip).

2. Unzip and place the resulting folder within your  WordPress Plugins directory (typically `wp-content/plugins`).

3. Activate the Plugin from within WP Admin.

Please note: you will not see any settings pages within WP Admin. This Plugin currently
requires you to call the image helper functions manually (see [Usage](#usage) below).]

### Requirements

- PHP >=5.4
- Fileinfo Extension

### Supported Image Libraries

- GD Library (>=2.0)
- Imagick PHP extension (>=6.5.7)

Please refer to the [Intervention Library](http://image.intervention.io/) for
full details.

## Usage

### Getting Started

- [Install the Plugin](#installation) and Activate.
- Use the Plugin in your Theme or Plugin.

### Basic Usage

WP Intervention exposes a global helper `wp_intervention` for use in your Theme.

The API for this is:

```
wp_intervention( string $src, array $intervention_args [, array $options] );
```

The first parameter (`$src`) is the source image to be processed. This can be either:

* a file path to an image (eg:
`~/public_html/web/wp-content/uploads/foo.jpg`).
* a publicly
accessible URL (eg: `https://example.com/wp-content/uploads/foo.jpg`).

### Intervention Arguments

The second `$intervention_args` parameter is an settings array which determines the transformations to be applied to the image.

WP Intervention supports the [same API as the underlying Intervention Library](http://image.intervention.io/). Indeed, under the hood the Plugin simply *proxies all arguments as method calls to PHP Intervention*.

In the associative settings array the key should be the Intervention transformation name (eg: `fit`, `blur`...etc) and the value should be the arguments to pass to that transformation.

#### Example Usage

```php
// Define a source image
$original_img = "..."; // source image file path or URL here

// An array of args to be *invoked* on Intervention
$intervention_args = array(

	// http://image.intervention.io/api/fit
	'fit' => array(
		'100',
		'600'
	),

	// http://image.intervention.io/api/blur
	'blur' => 20,

	// http://image.intervention.io/api/rotate
	'rotate' => -45
);

// Call the global helper function
$img_src = wp_intervention( $original_image, $intervention_args );
```

Methods are invoked on the underlying Intervention library in the *order defined in the arguments array*.

Refer to the [documentation](http://image.intervention.io/) to learn more about the capabilities of Intervention (hint: it's pretty fully featured!).

### Options

Note: these are not the image manipulation arguments passed to the Intervention Library.

There are also various options which can be modified by passing a 3rd argument to the global function on a per use case basis.

```php
wp_intervention( $original_image, $intervention_args, array(
	'quality' => 100,
	'cache'   => false, // be careful about modifying this (see below)
	'debug'   => true, // enable verbose debug mode
) );
```

#### Options Settings

##### quality

Type: Integer

Default: 80

Description: A image quality setting ranging from 0-100.

##### cache

Type: Boolean

Default: true

Description: Whether or not to cache this particular image. If false the image will be re-generated by Intervention on each request which may place undue load on your server. Use wisely...

##### debug

Type: Boolean

Default: value of `WP_DEBUG` constant or (if not defined) `false`.

Description: Enable verbose debugging mode. Will throw and enable exceptions which would otherwise be silently caught by the wrapper. Useful for debugging.

#### Overriding the Default Options

The default settings for the options (see above) can be overidden globally by filtering on the hook `wpi_default_options`. For example

```php
function modify_default_options( $options ) {
	$options['cache'] = false;
    return $options;
}
add_filter( 'wpi_default_options', 'modify_default_options' );
```

## Image Driver Configuration

Intervention allows you to configure either GD or Imagick as the primary image driver. By default this is GD but you can overide this by filtering on the `wpi_driver` hook.

```php
function modify_wpi_driver( $driver ) {
	$driver = 'imagick';
    return $driver;
}
add_filter( 'wpi_driver', 'modify_wpi_driver' );
```

Note if you pass an unsupported driver then this is your problem...

## Caching

WP Intervention will cache generated images on disk to avoid repeatedly processing images. By default the cache resides at:

`/wp-content/uploads/intervention/cache/`

Note: the exact path may vary depending on the settings of your individual WordPress installation.

### Altering the default cache location

If for any reason you need to change the default location of the cache you can filter on the `wpi_cache_directory` hook.

```php
function modify_wpi_cache_dir( $dir ) {
	$dir = '/some/random/location/you/need/to/set/';
    return $dir;
}
add_filter( 'wpi_cache_directory', 'modify_wpi_cache_dir' );
```

You are responsible for ensuring this path is writable by PHP and publicly accessible.

### Cache Garbage Collection

WP Intervention automatically removes cached images that have not been accessed for 24hrs.

To do this each time a cached image is accessed by WP Intervention it's timestamp is updated via PHP's `touch()` method. A WP Cron job then runs every hour checking for any "stale" images that are older than 24hrs. Any matching images are deleted.

If for any reason you need to alter the duration cached files are allowed to reside on disk before they become stale, then you can filter on the `wpi_clean_outdate_cache_files_period` hook.

```php
function modify_wpi_cache_files_period( $period ) {
	$period = 604800; // 1 week
    return $period;
}
add_filter( 'wpi_clean_outdate_cache_files_period', 'modify_wpi_cache_files_period' );
```

Similarly if you need to alter how often the cron job reoccurs then you can filter on `wpi_clean_outdate_cache_files_cron_recurrance`.

```php
function modify_wpi_clean_outdate_cache_files_cron_recurrance( $recurrance ) {
	$recurrance = 'twicedaily';
    return $recurrance;
}
add_filter( 'wpi_clean_outdate_cache_files_cron_recurrance', 'modify_wpi_clean_outdate_cache_files_cron_recurrance' );
```

## Contributing

Contributions to the WP Intervention Plugin are welcome. Please make a pull request!

## License

WP Intervention Plugin is licensed under the [MIT License](http://opensource.org/licenses/MIT).

Intervention Image is licensed under the [MIT License](http://opensource.org/licenses/MIT).
Copyright 2014 [Oliver Vogel](http://olivervogel.net/)