# Ansible Role: PHP

[![Build Status](https://travis-ci.org/geerlingguy/ansible-role-php.svg?branch=master)](https://travis-ci.org/geerlingguy/ansible-role-php)

Installs PHP on RedHat/CentOS and Debian/Ubuntu servers.

## Requirements

Must be running a separate web server, such as Nginx or Apache.

## Role Variables

Available variables are listed below, along with default values (see `vars/main.yml`):

    php_memory_limit: "256M"
    php_max_execution_time: "60"
    php_upload_max_filesize: "64M"

Some commonly-adjusted PHP ini directives. Adjust to suit your system.

    php_enable_apc: true

Whether to enable APC. Other APC variables will be ineffective if this is set to false.

    php_apc_enabled_in_ini: false

When installing APC, depending on the system and whether running PHP as a webserver module or standalone via `php-fpm`, you might need the line `extension=apc.so` in `apc.ini`. If you need that line added, set this variable to true.

    php_apc_cache_by_default: "1"
    php_apc_shm_size: "96M"

Two APC ini directives that are often customized on a system. Set `php_apc_cache_by_default` to 0 to disable APC by default (so you could enable it on a host-by-host basis). Set the `php_apc_shm_size` so it will hold all your application code in memory with a little overhead (fragmentation or APC running out of memory will slow down PHP dramatically).

This Ansible role assumes you're including `php-pecl-apc` in the list of `php_packages` below. It's rarely a good idea to run a PHP < 5.5 installation without some kind of opcode cache, and APC works great for PHP 5.3 and 5.4.

    php_date_timezone: "America/Chicago"

Explicitly set PHP's date timezone system-wide.

    php_sendmail_path: "/usr/sbin/sendmail -t -i"

The path to use for sendmail or a sendmail wrapper/replacement. You can also add options to this line if you need to set sendmail to use an explicit name/email for the sender.

    php_packages: []

A list of the PHP packages to install (OS-specific by default). You'll likely want to install common packages like `php`, `php-cli`, `php-devel` and `php-pdo`, and you can add in whatever other packages you'd like (for example, `php-gd` for image manipulation, or `php-ldap` if you need to connect to an LDAP server for authentication).

    php_enable_webserver: true

If your usage of PHP is tied to a web server (e.g. Apache or Nginx), leave this default value. If you are using PHP server-side or to run some small application, set this value to `false` so this role doesn't attempt to interact with a web server.

    php_webserver_daemon: "httpd"

The default values for the HTTP server deamon are `httpd` (used by Apache) for RedHat/CentOS, or `apache2` (also used by Apache) for Debian/Ubuntu. If you are running another webserver (for example, `nginx`), change this value to the name of the daemon under which the webserver runs.

    php_enable_php_fpm: false

If you add `php-fpm` to the `php_packages` list, and would like to run PHP-fpm, as you would with Nginx or as an alternative to `mod_php` in Apache, you can set this variable to `true`, and the `php-fpm` daemon will be enabled and started. You will need to configure PHP-fpm on your own, by editing the config file in `/etc/php-fpm.d/www.conf` (for RedHat servers) or replacing it with your own template via Ansible.

    php_enablerepo: ""

(RedHat/CentOS only) If you have enabled any additional repositories (might I suggest geerlingguy.repo-epel or geerlingguy.repo-remi), those repositories can be listed under this variable (e.g. `remi,epel`). This can be handy, as an example, if you want to install the latest version of PHP 5.4, which is in the Remi repository.

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
