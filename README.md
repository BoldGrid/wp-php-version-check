# wp-php-version-check
Composer package to verify that WordPress and PHP versions satisfy a minimum version requirement.

---

### Jump To Section
- [Why?](#why)
- [What Does It Do?](#what-does-it-do)
- [Installation](#installation)
- [Usage](#usage)
- [Example](#example)

---

### Why?

I haven't really seen any reliable packages that perform this functionality whenever I needed it, so I figured I might as well make my own for future use cases.  Most of the ones I found don't deactivate the plugin if the version requirements aren't met, they leave the success admin notice up after activation with the error admin notice, they don't support WP-CLI activation cleanly, and only check either the PHP version or the WordPress version instead of both.  A special thank you to Dan Bissonnet's [wp-version-check](https://github.com/dbisso/wp-version-check) as it was an inspiration for this, but unfortunately hasn't been updated for 3+ years.  More than anything though - I wanted to create a package that does what I need, and assume someone else out there might be looking for the same! :)

---

### What does it do?

If an environment does not meet the minimum PHP or WordPress version requirements, the plugin is deactivated and an error admin notice is shown in the admin dashboard.  The notice informs the user of the minimum requirements, and their current versions of WordPress and PHP so they can change their environment to satisfy the plugin's version requirements.  If the user activates the plugin through WP-CLI then a WP-CLI Warning is displayed informing them of the same information.  The plugin is then able to run it's code in the WordPress init hook using the generated hook name, or it call a callable method to run it's initialization code if init is not desirable.

---

### Installation

Using composer, you can get started quickly:

```php
$ composer require timelsass/wp-php-version-check
```

---

### Usage

We aren't going to be using the autoloader for this package because WordPress supports PHP 5.2 as a minimum, and this is just a single class.  In your code you would add to your main plugin file after the headers, something like this:

```php
if ( ! class_exists( 'Wp_Php_Version_Check' ) ) {
  require plugin_dir_path( __FILE__ ) . 'vendor/timelsass/wp-php-version-check/class-wp-php-version-check.php';
}
```

The init method accepts four parameters:

```php
Wp_Php_Version_Check::init( $plugin, $wp_version, $php_version (, $callback, ...$args) );
```

1. `$plugin`      - The main plugin file relative to the plugins directory.  Usually you would use: `plugin_basename( __FILE__ )`.
2. `$wp_version`  - The minimum WordPress version required for your plugin to work.
3. `$php_version` - The minimum PHP version required for your plugin to work.
4. `$callback`    - (Optional) A callable method.  The format should be like all other callables.
5. `...$args`     - (Optional) Any number of arguments to pass to the callable method. `$arg1, $arg2, $arg3` etc.

You simply need to initialize the version checking with the necessary parameters, like this:

```php
Wp_Php_Version_Check::init( plugin_basename( __FILE__ ), '4.7', '5.3' );
```

This will do the version checking portion of things for you, so your next step would be to add a hook telling WordPress what to do to initialize your plugin's actual code.

The hook name is dynamically generated based on $plugin.  If you have a plugin called "example-plugin", then the hook generated would be "example-plugin:init".

This code shows how to initalize some code for a plugin called "example-plugin" in an anonymous function (anonymous function passed on a hook requires PHP 5.3+):

```php
add_action( 'example-plugin:init', function() {
  wp_die( 'Passes Version Requirements.' );
});
```

It may not be desirable to add your initialization code to the init hook, so an optional parameter, `$callback`, is provided.  The callback expects a callable method to be used, or it will default back to adding the hook to init.  The callable format is the same as other WordPress callbacks/callable methods in general, and additional arguments are passed by including them after the callback argument.

This example shows how you would implement a callback with 2 required arguments:

```php
Wp_Php_Version_Check::init( plugin_basename( __FILE__ ), '4.7', '5.3', 'example_plugin_init', true, 'The plugin has been initialized!' );

function example_plugin_init( $passed, $message ) {
    if ( $passed ) {
        wp_die( $message );
    }
}
```

---

### Example

Here's a full example that you can use showing a version check for WordPress 4.0 and PHP 5.6 and running the plugin initialization code on the WordPress init hook:

```php
/**
 * Plugin Name: Example Plugin
 * Plugin URI: https://github.com/timelsass/wp-php-version-check
 * Description: This is an example of using Wp_Php_Version_Check class.
 * Version: 1.0.0
 * License: GPLv2 or later
 */
 
// Prevent direct calls.
defined( 'WPINC' ) ? : die;

// Load the Wp_Php_Version_Check file and make sure that the class doesn't exist first.
if ( ! class_exists( 'Wp_Php_Version_Check' ) ) {
  require plugin_dir_path( __FILE__ ) . 'vendor/timelsass/wp-php-version-check/class-wp-php-version-check.php';
}

// Initalize the version checking.  This checks that the user has at least WordPress v4.0 and PHP v5.6.
Wp_Php_Version_Check::init( plugin_basename( __FILE__ ), '4.0', '5.6' );

// Initalize our core plugin functionality in the example-plugin:init hook.
add_action( 'example-plugin:init', 'example_plugin_load' );

/**
 * Kicks off our core plugin code.
 */
function example_plugin_load() {
  wp_die( 'Example plugin passes the version check requirements! Woohoo!' );
}
```
