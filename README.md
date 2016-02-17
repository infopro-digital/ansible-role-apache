# apache

This role install and configure Apache under various OS

## Requirements

* A compatible host
  * Debian 8 or greater
  * FreeBSD 9.3 or greater
* Apache 2.4

## Role Variables

### Default variables
You will find here the default variables present for this role

```
---
apache:
  service_manage: true
  service_enable: true
  server_admin: admin@example.org
  ip: "*"
  port: 80
  errordocs: {}
  vhosts: []
  modules: []
  modules_blacklist: []
  jk:
    method: 'Busyness'
    workers: {}
  security:
    server_tokens: 'Prod'
    server_signature: 'Off'
    trace_enable: 'Off'
    protect_vcs_directories: true
    prevent_clickjacking: false

```

### Managing Apache modules

You can use apache.modules variables to enable modules and apache.modules_blacklist for disable modules.

Warning: On Debian, default installed modules are not modified by this playbook.

```
---
apache:
  modules:
    - deflate
    - jk
    - mime
    - mpm_worker
    - negotiation
    - reqtimeout
    - rewrite

  modules_blacklist:
    - cgid
    - mpm_event
    - status
```

The following modules will trigger a package installation:
* mod_jk
* mod_auth_pgsql

Other modules installation aren't support at this moment, don't hesistate to do a PR

### Managing Apache virtualhosts

You can manage your virtualhosts using the apache.vhosts variable.

Here is the exhaustive list of config variables:

* ip (optionnal, default apache.ip): apache listening IP
* port (optionnal, default apache.port): apache listening port
* server_admin (optionnal, default apache.server_admin): vhost server admin
* server_name: vhost servername
* document_root (optionnal, default /var/www/<vhost_name>)
* custom_log_format (optionnal, default combined): log output format for CustomLog
* indexes (optionnal, default -indexes): Document root indexes
* allowed_hosts (optionnal, default undefined): List of allowed IP who can use this DocumentRoot, if specified, else everybody
* allow_override (optionnal, default None): Allow overriding Apache configuration for DocumentRoot Directory
* document_root_fragment (optionnal, default ''): A custom raw apache configuration for DocumentRoot Directory
* rewrite_rules (optionnal, default undefined): List of Rewrite Rules to apply on this vhost with the following item attributes. Applied only if mod_rewrite is enabled

  * condition (optionnal): Rewrite rule condition
  * pattern: Pattern to match
  * dest: Rule to apply

* disallowed_path_regex (optionnal, default undefined): List of forbidden paths to protect. This uses a rewrite rule and a regex matching. Applied only if mod_rewrite is enabled
* disallowed_files (optionnal, default undefined): List of forbidden files to protect.
* deflate_compression_level (optionnal, default 9): mod_deflate compression level. Applied only if mod_deflate is enabled
* deflate_by_type (optionnal, default undefined): List of mime types to compress with deflate. Applied only if mod_deflate is enabled.
* expire_by_type (optionnal, default undefined): List of file types with a rule expire rule. Applied only if mod_expires is enabled
* proxy_preserve_host (optionnal, default Off): Preserve requested host when calling the backend server
* proxy_pass (optionnal, default undefined): List of reverse proxy objects with the following parameters. Applied only if mod_proxy is enabled

  * path: local path to map
  * url: backend URL to call

```
---
apache:
  vhosts:
    - name: "wordpress_front"
      server_admin: "no-reply@crazy-app.example"
      server_name: "wordpress_front.crazy-app.example"
      deflate_by_type:
        - image/*
        - text/css
      expire_by_type:
        'image/png':
          rule: "access plus 1 day"
```

### Managing JK workers

You can manage your mod_jk workers using the apache.jk.workers variable. 
This variable needs a list of workers and each worker awaits a list of nodes

If you want to change the load balancing method, you can change globally the apache.jk.method (default is Busyness) 
or per worker using the method attribute in your worker
```
---
apache:
  jk:
    workers:
      group_amazing_app:
        nodes:
          io_tomcat_01: { ip: 10.0.0.1 }
          io_tomcat_02: { ip: 10.0.0.2 }
```

### Managing apache security

By default this module configure some apache security. You can configure apache.security keys to set some global security values

* server_tokens (default: Prod): OS type & compiled modules
* server_signature (default: Off): Server version & virtualhost name
* trace_enable (default: Off)
* prevent_clickjacking (default: false): add X-Frame-Options: "sameorigin" header to prevent content embedded on other sites
* protect_vcs_directories (default: true): forbid access to .svn and .git directories
```
---
apache:
  security:
    server_tokens: 'Prod'
    server_signature: 'Off'
    trace_enable: 'Off'
    prevent_clickjacking: false
    protect_vcs_directories: true
```

## Dependencies

Nothing

## Example Playbook

    - hosts: apache_aws_cluster
      vars_files: vars/apache_aws_cluster.yml
      roles:
         - infopro-digital.apache

## License

BSD

## Author Information

Created by Loic Blot <loic.blot@unix-experience.fr> (http://www.unix-experience.fr)
Sponsored by Infopro Digital (http://www.infopro-digital.com/) & E.T.A.I. (http://www.etai.fr)
