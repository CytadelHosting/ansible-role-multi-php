Ansible Role: Multi PHP
=========

Ansible role providing multiple PHP versions on Debian.

# Ansible Role: [CytadelHosting.multi-php](https://github.com/CytadelHosting/ansible-role-multi-php)

Role Variables
--------------

Default PHP version:

```yaml
php_default_version: "php7.3"
```

List of PHP modules to be installed across all versions:

```yaml
php_default_modules:
 - bcmath
 - bz2
 - curl
 - gd
 - readline
 - json
 - mysql
 - soap
 - opcache
 - xml
 - xmlrpc
 - zip
```

Can be specialized by:

```yaml
php56_default_modules: "{{ php_default_modules }}"
php70_default_modules: "{{ php_default_modules }}"
php71_default_modules: "{{ php_default_modules }}"
php72_default_modules: "{{ php_default_modules }}"
php73_default_modules: "{{ php_default_modules }}"
php74_default_modules: "{{ php_default_modules }}"
php80_default_modules: "{{ php_default_modules }}"
php81_default_modules: "{{ php_default_modules }}"
```

One special super variable:

```yaml
php: []
```

### PHP-FPM

PHP-FPM is a simple and robust FastCGI Process Manager for PHP. It can dramatically ease scaling of PHP apps and is the normal way of running PHP-based sites and apps when using a webserver like Nginx (though it can be used with other webservers just as easily).

When using this role with PHP running as `php-fpm` instead of as a process inside a webserver (e.g. Apache's `mod_php`), you need to set the following variable to `true`:

    php_enable_php_fpm: false

If you're using Apache, you can easily get it configured to work with PHP-FPM using the [geerlingguy.apache-php-fpm](https://github.com/geerlingguy/ansible-role-apache-php-fpm) role.

    php_fpm_state: started
    php_fpm_enabled_on_boot: true

Control over the fpm daemon's state; set these to `stopped` and `false` if you want FPM to be installed and configured, but not running (e.g. when installing in a container).

    php_fpm_handler_state: restarted

The handler restarts PHP-FPM by default. Setting the value to `reloaded` will reload the service, intead of restarting it.


    php_fpm_pools:
      - pool_name: www
        pool_template: www.conf.j2
        pool_listen: "127.0.0.1:9000"
        pool_listen_allowed_clients: "127.0.0.1"
        pool_pm: dynamic
        pool_pm_max_children: 5
        pool_pm_start_servers: 2
        pool_pm_min_spare_servers: 1
        pool_pm_max_spare_servers: 3
        pool_pm_max_requests: 500

List of PHP-FPM pool to create. By default, www pool is created. To setup a new pool, add an item to php_fpm_pools list.

Specific settings inside the default `www.conf.j2` PHP-FPM pool. If you'd like to manage additional settings, you can do so either by replacing the file with your own template using `pool_template`.

### php.ini settings

    php_use_managed_ini: true

By default, all the extra defaults below are applied through the php.ini included with this role. You can self-manage your php.ini file (if you need more flexility in its configuration) by setting this to `false` (in which case all the below variables will be ignored).

    php_fpm_pool_user: "[apache|nginx|other]" # default varies by OS
    php_fpm_pool_group: "[apache|nginx|other]" # default varies by OS
    php_memory_limit: "256M"
    php_max_execution_time: "60"
    php_max_input_time: "60"
    php_max_input_vars: "1000"
    php_realpath_cache_size: "32K"
    php_file_uploads: "On"
    php_upload_max_filesize: "64M"
    php_max_file_uploads: "20"
    php_post_max_size: "32M"
    php_date_timezone: "America/Chicago"
    php_allow_url_fopen: "On"
    php_sendmail_path: "/usr/sbin/sendmail -t -i"
    php_output_buffering: "4096"
    php_short_open_tag: false
    php_error_reporting: "E_ALL & ~E_DEPRECATED & ~E_STRICT"
    php_display_errors: "Off"
    php_display_startup_errors: "On"
    php_expose_php: "On"
    php_session_cookie_lifetime: 0
    php_session_gc_probability: 1
    php_session_gc_divisor: 1000
    php_session_gc_maxlifetime: 1440
    php_session_save_handler: files
    php_session_save_path: ''
    php_disable_functions: []
    php_precision: 14
    php_serialize_precision: "-1"

Various defaults for PHP. Only used if `php_use_managed_ini` is set to `true`.

### OpCache-related Variables

The OpCache is included in PHP starting in version 5.5, and the following variables will only take effect if the version of PHP you have installed is 5.5 or greater.

    php_opcache_zend_extension: "opcache.so"
    php_opcache_enable: "1"
    php_opcache_enable_cli: "0"
    php_opcache_memory_consumption: "96"
    php_opcache_interned_strings_buffer: "16"
    php_opcache_max_accelerated_files: "4096"
    php_opcache_max_wasted_percentage: "5"
    php_opcache_validate_timestamps: "1"
    php_opcache_revalidate_path: "0"
    php_opcache_revalidate_freq: "2"
    php_opcache_max_file_size: "0"

OpCache ini directives that are often customized on a system. Make sure you have enough memory and file slots allocated in the OpCache (`php_opcache_memory_consumption`, in MB, and `php_opcache_max_accelerated_files`) to contain all the PHP code you are running. If not, you may get less-than-optimal performance!

For custom opcache.so location provide full path with `php_opcache_zend_extension`.

    php_opcache_conf_filename: [platform-specific]

The platform-specific opcache configuration filename. Generally the default should work, but in some cases, you may need to override the filename.

### APCu-related Variables

    php_enable_apc: true

Whether to enable APCu. Other APCu variables will be ineffective if this is set to false.

    php_apc_shm_size: "96M"
    php_apc_enable_cli: "0"

APCu ini directives that are often customized on a system. Set the `php_apc_shm_size` so it will hold all cache entries in memory with a little overhead (fragmentation or APC running out of memory will slow down PHP *dramatically*).

    php_apc_conf_filename: [platform-specific]

The platform-specific APC configuration filename. Generally the default should work, but in some cases, you may need to override the filename.

#### Ensuring APC is installed

If you use APC, you will need to make sure APC is installed (it is installed by default, but if you customize the `php_packages` list, you need to include APC in the list):

  - *On RHEL/CentOS systems*: Make sure `php-pecl-apcu` is in the list of `php_packages`.
  - *On Debian/Ubuntu systems*: Make sure `php-apcu` is in the list of `php_packages`.


This variable controls installed PHP versions, modules and is used to configure PHP settings. See examples below.


Dependencies
------------

None.

Example Playbook
----------------

Example 1: Just install PHP 5.6 and 7.3

```yaml
- hosts: php
  roles:
     - Cytadel.multi-php.php
  vars:
    php:
      - version: php56
      - version: php73
```

Example 2: Install PHP 5.6, 7.3, 8.0 with extra modules and configures global CLI/FPM settings.

```yaml
- hosts: php
  roles:
     - Cytadel.multi-php.php
  vars:
    # local variable, see below
    common_fpm_ini:
     - { "section": "PHP",     "key": "max_execution_time",             "value": "360"  }
     - { "section": "PHP",     "key": "post_max_size",                  "value": "64MB" }
     - { "section": "PHP",     "key": "upload_max_filesize",            "value": "64MB" }
     - { "section": "opcache", "key": "opcache.enable",                 "value": "1"    }
     - { "section": "opcache", "key": "opcache.memory_consumption",     "value": "512"  }
     - { "section": "opcache", "key": "opcache.max_accelerated_files",  "value": "500"  }
    php:
     - version: php56
       fpm_ini: "{{ common_fpm_ini }}"
       cli_ini:
        - section: "PHP"
          key: "memory_limit"
          value: "2048M"
     - version: php73
       modules_extra: ['pgsql', 'gd']
       fpm_ini: "{{ common_fpm_ini }}"
     - version: php80
       modules_extra: ['pgsql', 'gd']
       fpm_ini: "{{ common_fpm_ini }}"
```

## License

BSD

## Author Information

This role was first created in 2023 by [Ahmed Trabelsi](https://github.com/atrabelsiJetpulp) from [Lukáš Kasič](https://github.com/lukasic).
Added special configuration for php-fpm, like pool, opcache, apcu from [Jeff Geerling](https://github.com/geerlingguy).

[Cytadel](https://www.cytadel.fr/) is a Hosting Company based in Lyon, France.

Cytadel is a business unit of [JETPULP](https://www.jetpulp.fr/), and is a member of [Altavia Group](https://www.altavia.com/).