# Ansible Role: PHP

[![Build Status](https://travis-ci.org/geerlingguy/ansible-role-php.svg?branch=master)](https://travis-ci.org/geerlingguy/ansible-role-php)

Installs PHP on RedHat/CentOS and Debian/Ubuntu servers.

## Requirements

Must be running a separate web server, such as Nginx or Apache.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    php_packages: []

A list of the PHP packages to install (OS-specific by default). You'll likely want to install common packages like `php`, `php-cli`, `php-devel` and `php-pdo`, and you can add in whatever other packages you'd like (for example, `php-gd` for image manipulation, or `php-ldap` if you need to connect to an LDAP server for authentication).

    php_enable_webserver: true

If your usage of PHP is tied to a web server (e.g. Apache or Nginx), leave this default value. If you are using PHP server-side or to run some small application, set this value to `false` so this role doesn't attempt to interact with a web server.

    php_webserver_daemon: "httpd"

The default values for the HTTP server deamon are `httpd` (used by Apache) for RedHat/CentOS, or `apache2` (also used by Apache) for Debian/Ubuntu. If you are running another webserver (for example, `nginx`), change this value to the name of the daemon under which the webserver runs.

    php_enablerepo: ""

(RedHat/CentOS only) If you have enabled any additional repositories (might I suggest geerlingguy.repo-epel or geerlingguy.repo-remi), those repositories can be listed under this variable (e.g. `remi,epel`). This can be handy, as an example, if you want to install the latest version of PHP 5.4, which is in the Remi repository.

### PHP-FPM

PHP-FPM is a simple and robust FastCGI Process Manager for PHP. It can dramatically ease scaling of PHP apps and is the normal way of running PHP-based sites and apps when using a webserver like Nginx (though it can be used with other webservers just as easily).

When using this role with PHP running as `php-fpm` instead of as a process inside a webserver (e.g. Apache's `mod_php`), you need to set the following variable to `true`:

    php_enable_php_fpm: false

You will also need to override the default `php_packages` list and add `php-fpm` (RedHat/CentOS) or `php5-fpm` (Debian/Ubuntu) to the list.

This role does not manage fpm-specific www pool configuration (found in `/etc/php-fpm.d/www.conf` on RedHat/CentOS and `/etc/php5/fpm/pool.d/www.conf` on Debian/Ubuntu), but rather allows you to manage those files on your own. If you change that file, remember to notify the `restart php-fpm` handler so PHP picks up the new settings once in place. Settings like `pm.max_children` and other `pm.*` settings can have a dramatic impact on server performance, and should be tuned specifically for each application and server configuration.

### php.ini settings

    php_use_managed_ini: true

By default, all the extra defaults below are applied through the php.ini included with this role. You can self-manage your php.ini file (if you need more flexility in its configuration) by setting this to `false` (in which case all the below variables will be ignored).

    php_memory_limit: "256M"
    php_max_execution_time: "60"
    php_realpath_cache_size: "32K"
    php_upload_max_filesize: "64M"
    php_date_timezone: "America/Chicago"
    php_sendmail_path: "/usr/sbin/sendmail -t -i"
    php_short_open_tag: false
    php_error_reporting: "E_ALL & ~E_DEPRECATED & ~E_STRICT"
    php_display_errors: "Off"

Various defaults for PHP. Only used if `php_use_managed_ini` is set to `true`.

### APC-related Variables

    php_enable_apc: true

Whether to enable APC. Other APC variables will be ineffective if this is set to false.

    php_apc_enabled_in_ini: false

When installing APC, depending on the system and whether running PHP as a webserver module or standalone via `php-fpm`, you might need the line `extension=apc.so` in `apc.ini`. If you need that line added (e.g. you're running `php-fpm`), set this variable to true.

    php_apc_cache_by_default: "1"
    php_apc_shm_size: "96M"
    php_apc_stat: "1"
    php_apc_enable_cli: "0"

APC ini directives that are often customized on a system. Set `php_apc_cache_by_default` to 0 to disable APC by default (so you could enable it on a host-by-host basis). Set the `php_apc_shm_size` so it will hold all your application code in memory with a little overhead (fragmentation or APC running out of memory will slow down PHP *dramatically*).

#### Ensuring APC is installed

If you use APC, you will need to make sure APC is installed (it is installed by default, but if you customize the `php_packages` list, you need to include APC in the list):

  - *On RHEL/CentOS systems*: Make sure `php-pecl-apc` is in the list of `php_packages`.
  - *On Debian/Ubuntu systems*: Make sure `php-apc` is in the list of `php_packages`.

You can also install APC via `pecl`, but it's simpler to manage the installation with the system's package manager.

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

## TODO

  - Make role more flexible, allowing APC to be excluded from `php_packages` list.
  - Use `lineinfile` rather than templates to make configuration changes.

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
