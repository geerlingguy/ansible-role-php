# Ansible Role: PHP

Installs PHP on RedHat/CentOS and Debian/Ubuntu servers.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `vars/main.yml`):

    php_memory_limit: "256M"
    php_max_execution_time: "60"
    php_upload_max_filesize: "64M"

Some commonly-adjusted PHP ini directives. Adjust to suit your system.

    php_apc_cache_by_default: "1"
    php_apc_shm_size: "96M"

Two APC ini directives that are often customized on a system. Set `php_apc_cache_by_default` to 0 to disable APC by default (so you could enable it on a host-by-host basis). Set the `php_apc_shm_size` so it will hold all your application code in memory with a little overhead (fragmentation or APC running out of memory will slow down PHP dramatically).

This Ansible role assumes you're including `php-pecl-apc` in the list of `php_packages` below. It's rarely a good idea to run a PHP < 5.5 installation without some kind of opcode cache, and APC works great for PHP 5.3 and 5.4.

    php_date_timezone: "America/Chicago"

Explicitly set PHP's date timezone system-wide.

    php_packages: []

A list of the PHP packages to install (OS-specific by default). You'll likely want to install common packages like `php`, `php-cli`, `php-devel` and `php-pdo`, and you can add in whatever other packages you'd like (for example, `php-gd` for image manipulation, or `php-ldap` if you need to connect to an LDAP server for authentication).

    php_enablerepo: ""

(RedHat/CentOS only) If you have enabled any additional repositories (might I suggest geerlingguy.repo-epel or geerlingguy.repo-remi), those repositories can be listed under this variable (e.g. `remi,epel`). This can be handy, as an example, if you want to install the latest version of PHP 5.4, which is in the Remi repository.

## Dependencies

  - geerlingguy.apache

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
  - Remove apache dependency (better way of using handler? Way to get around Debian/apt's apache2 + php bindings?).

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
