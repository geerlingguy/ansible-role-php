# Ansible Role: PHP

[![Build Status](https://travis-ci.org/geerlingguy/ansible-role-php.svg?branch=master)](https://travis-ci.org/geerlingguy/ansible-role-php)

Installs PHP on RedHat/CentOS, Debian/Ubuntu, & FreeBSD servers.

## Requirements

Must be running a separate web server, such as Nginx or Apache.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    php_packages: []

A list of the PHP packages to install (OS-specific by default). You'll likely want to install common packages like `php`, `php-cli`, `php-devel` and `php-pdo`, and you can add in whatever other packages you'd like (for example, `php-gd` for image manipulation, or `php-ldap` if you need to connect to an LDAP server for authentication).

_Note: If you're using Debian/Ubuntu, you may also need to install `libapache2-mod-fastcgi` (for cgi/PHP-FPM) or `libapache2-mod-php5` (or a similar package depending on PHP version) if you want to use `mod_php` with Apache._

    php_enable_webserver: true

If your usage of PHP is tied to a web server (e.g. Apache or Nginx), leave this default value. If you are using PHP server-side or to run some small application, set this value to `false` so this role doesn't attempt to interact with a web server.

    php_webserver_daemon: "httpd"

The default values for the HTTP server deamon are `httpd` (used by Apache) for RedHat/CentOS, or `apache2` (also used by Apache) for Debian/Ubuntu. If you are running another webserver (for example, `nginx`), change this value to the name of the daemon under which the webserver runs.

    php_enablerepo: ""

(RedHat/CentOS only) If you have enabled any additional repositories (might I suggest [geerlingguy.repo-epel](https://github.com/geerlingguy/ansible-role-repo-epel) or [geerlingguy.repo-remi](https://github.com/geerlingguy/ansible-role-repo-remi)), those repositories can be listed under this variable (e.g. `remi-php56,epel`). This can be handy, as an example, if you want to install the latest version of PHP 5.6, which is in the Remi repository.

    php_executable: "php"

The executable to run when calling PHP from the command line. You should only change this if running `php` on your server doesn't target the correct executable, or if you're using software collections on RHEL/CentOS and need to target a different version of PHP.

### PHP-FPM

PHP-FPM is a simple and robust FastCGI Process Manager for PHP. It can dramatically ease scaling of PHP apps and is the normal way of running PHP-based sites and apps when using a webserver like Nginx (though it can be used with other webservers just as easily).

When using this role with PHP running as `php-fpm` instead of as a process inside a webserver (e.g. Apache's `mod_php`), you need to set the following variable to `true`:

    php_enable_php_fpm: false

If you're using Apache, you can easily get it configured to work with PHP-FPM using the [geerlingguy.apache-php-fpm](https://github.com/geerlingguy/ansible-role-apache-php-fpm) role.

    php_fpm_listen: "127.0.0.1:9000"
    php_fpm_listen_allowed_clients: "127.0.0.1"
    php_fpm_pm_max_children: 50
    php_fpm_pm_start_servers: 5
    php_fpm_pm_min_spare_servers: 5
    php_fpm_pm_max_spare_servers: 5

Specific settings inside the default `www.conf` PHP-FPM pool. If you'd like to manage additional settings, you can do so either by replacing the file with your own template or using `lineinfile` like this role does inside `tasks/configure.yml`.

### php.ini settings

    php_use_managed_ini: true

By default, all the extra defaults below are applied through the php.ini included with this role. You can self-manage your php.ini file (if you need more flexility in its configuration) by setting this to `false` (in which case all the below variables will be ignored).

    php_memory_limit: "256M"
    php_max_execution_time: "60"
    php_max_input_time: "60"
    php_max_input_vars: "1000"
    php_realpath_cache_size: "32K"
    php_upload_max_filesize: "64M"
    php_post_max_size: "32M"
    php_date_timezone: "America/Chicago"
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

Various defaults for PHP. Only used if `php_use_managed_ini` is set to `true`.

### OpCache-related Variables

The OpCache is included in PHP starting in version 5.5, and the following variables will only take effect if the version of PHP you have installed is 5.5 or greater.

    php_opcache_enabled_in_ini: false

When installing Opcache, depending on the system and whether running PHP as a webserver module or standalone via `php-fpm`, you might need the line `extension=opcache.so` in `opcache.ini`. If you need that line added (e.g. you're running `php-fpm`), set this variable to true.

    php_opcache_enable: "1"
    php_opcache_enable_cli: "0"
    php_opcache_memory_consumption: "96"
    php_opcache_interned_strings_buffer: "16"
    php_opcache_max_accelerated_files: "4096"
    php_opcache_max_wasted_percentage: "5"
    php_opcache_validate_timestamps: "1"
    php_opcache_revalidate_freq: "2"
    php_opcache_max_file_size: "0"

OpCache ini directives that are often customized on a system. Make sure you have enough memory and file slots allocated in the OpCache (`php_opcache_memory_consumption`, in MB, and `php_opcache_max_accelerated_files`) to contain all the PHP code you are running. If not, you may get less-than-optimal performance!

    php_opcache_conf_filename: [platform-specific]

The platform-specific opcache configuration filename. Generally the default should work, but in some cases, you may need to override the filename.

### APC-related Variables

    php_enable_apc: true

Whether to enable APC. Other APC variables will be ineffective if this is set to false.

    php_apc_enabled_in_ini: false

When installing APC, depending on the system and whether running PHP as a webserver module or standalone via `php-fpm`, you might need the line `extension=apc.so` in `apc.ini`. If you need that line added (e.g. you're running `php-fpm`), set this variable to true.

    php_apc_cache_by_default: "1"
    php_apc_shm_size: "96M"
    php_apc_stat: "1"
    php_apc_enable_cli: "0"

APC ini directives that are often customized on a system. Set `php_apc_cache_by_default` to 0 to disable APC by default (so you could just enable it for one codebase if you have a *lot* of code on a server). Set the `php_apc_shm_size` so it will hold all your application code in memory with a little overhead (fragmentation or APC running out of memory will slow down PHP *dramatically*).

    php_apc_conf_filename: [platform-specific]

The platform-specific APC configuration filename. Generally the default should work, but in some cases, you may need to override the filename.

#### Ensuring APC is installed

If you use APC, you will need to make sure APC is installed (it is installed by default, but if you customize the `php_packages` list, you need to include APC in the list):

  - *On RHEL/CentOS systems*: Make sure `php-pecl-apc` is in the list of `php_packages`.
  - *On Debian/Ubuntu systems*: Make sure `php-apc` is in the list of `php_packages`.

You can also install APC via `pecl`, but it's simpler to manage the installation with the system's package manager.

### Installing from Source

If you need a specific version of PHP, or would like to test the latest (e.g. master) version of PHP, there's a good chance there's no suitable package already available in your platform's package manager. In these cases, you may choose to install PHP from source by compiling it directly.

Note that source compilation takes *much* longer than installing from packages (PHP HEAD takes 5+ minutes to compile on a modern quad-core computer, just as a point of reference).

    php_install_from_source: false

Set this to `true` to install PHP from source instead of installing from packages.

    php_source_version: "master"

The version of PHP to install from source (a git branch, tag, or commit hash).

    php_source_clone_dir: "~/php-src"
    php_source_install_path: "/opt/php"
    php_source_install_gmp_path: "/usr/include/x86_64-linux-gnu/gmp.h"

Location where source will be cloned and installed, and the location of the GMP header file (which can be platform/distribution specific).

    php_source_make_command: "make"

Set the `make` command to `make --jobs=X` where `X` is the number of cores present on the server where PHP is being compiled. Will speed up compilation times dramatically if you have multiple cores.

    php_source_configure_command: >
      [...]

The `./configure` command that will build the Makefile to be used for PHP compilation. Add in all the options you need for your particular environment. Using a folded scalar (`>`) allows you to define the variable over multiple lines, which is extremely helpful for legibility and source control!

A few other notes/caveats for specific configurations:

  - **Apache with `mpm_prefork`**: If you're using Apache with prefork as a webserver for PHP, you will need to make sure `apxs2` is available on your system (e.g. by installing `apache2-prefork-dev` in Ubuntu), and you will need to make sure the option `--with-apxs2` is defined in `php_source_configure_command`. Finally, you will need to make sure the `mpm_prefork` module is loaded instead of `mpm_worker` or `mpm_event`, and likely add a `phpX.conf` (where `X` is the major version of PHP) configuration file to the Apache module config folder with contents like [`php7.conf`](https://gist.github.com/geerlingguy/5ae5445f28e71264e8c1).
  - **Apache with `mpm_event` or `mpm_worker`**: If you're using Apache with event or worker as a webserver for PHP, you will need to compile PHP with FPM. Make sure the option `--enable-fpm` is defined in `php_source_configure_command`. You'll also need to make sure Apache's support for CGI and event is installed (e.g. by installing `apache2-mpm-event` and `libapache2-mod-fastcgi`) and the `mpm_event` module is loaded.
  - **Nginx**: If you're using Nginx as a webserver for PHP, you will need to compile PHP with FPM. Make sure the option `--enable-fpm` is defined in `php_source_configure_command`.

## Dependencies

None.

## Example Playbook

    - hosts: webservers
      vars_files:
        - vars/main.yml
      roles:
        - { role: geerlingguy.php }

*Inside `vars/main.yml`*:

    php_memory_limit: "128M"
    php_max_execution_time: "90"
    php_upload_max_filesize: "256M"
    php_packages:
      - php
      - php-cli
      - php-common
      - php-devel
      - php-gd
      - php-mbstring
      - php-pdo
      - php-pecl-apc
      - php-xml
      ...

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
