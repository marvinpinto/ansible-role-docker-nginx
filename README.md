docker-nginx
============

[![Build Status](https://travis-ci.org/marvinpinto/ansible-role-docker-nginx.svg?branch=master)](https://travis-ci.org/marvinpinto/ansible-role-docker-nginx)

Ansible role to manage and run the nginx docker container.

Requirements
------------

This role has only been tested on Ubuntu 14.04. Since this uses Ansible's
docker module, you will need to ensure that a recent-ish version of `docker-py`
and `docker` are installed.

Examples
--------

Install this module from Ansible Galaxy into the './roles' directory:

```bash
ansible-galaxy install marvinpinto.docker-nginx -p ./roles
```

Use it in a playbook as follows, assuming you already have docker setup:

```yaml
- hosts: 'servers'
  roles:
    - role: 'marvinpinto.docker-nginx'
      become: yes
      nginx_conf: |
        user root;
        worker_processes 1;

        error_log /var/log/nginx/error.log warn;
        pid /var/run/nginx.pid;

        events {
            worker_connections  1024;
        }

        http {
            include       /etc/nginx/mime.types;
            default_type  application/octet-stream;
            log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                              '$status $body_bytes_sent "$http_referer" '
                              '"$http_user_agent" "$http_x_forwarded_for"';
            access_log  /var/log/nginx/access.log  main;
            sendfile        on;
            keepalive_timeout  65;
            include /etc/nginx/conf.d/*.conf;
        }
```

Have a look at the [defaults/main.yml](defaults/main.yml) for role variables
that can be overridden! If you need a playbook to set Docker itself, have a
look at
[marvinpinto.docker](https://github.com/marvinpinto/ansible-role-docker) Galaxy
role.

Expected to Be Configured
-------------------------

* `nginx_reverse_proxy_proxies`:  list of reverse proxy configurations; each configuration needs the following variables
  * `nginx_reverse_proxy_backend_name:` string, name nginx config uses to refer to the backend
  * `nginx_reverse_proxy_domains`: list of public-facing domains to be proxied
  * `nginx_reverse_proxy_backends`: list of backend servers, including ports and [other valid parameters for `server` in the `upstream` context of an nginx config file](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server)
  * `nginx_reverse_proxy_config_name`: name to use for the proxy file (do not include the '.conf' extension, role will add this)

Example Playbook
----------------

```yaml
---
# file group_vars/nginx_docker_proxy

nginx_reverse_proxy_proxies:
  - config_name: app1proxy
    backend_name: my-backend-1
    backends:
      - localhost:1880 weight=2
      - localhost:1881
    domains:
      - app1.192.168.88.10.xip.io
  - config_name: app2proxy
    backend_name: my-backend-2
    backends:
      - localhost:1882
      - localhost:1883
    domains:
      - app2.192.168.88.10.xip.io
    balancer_config: least_conn;

```

License
-------

[MIT](LICENSE.txt)

Author Information
------------------

* Marvin Pinto

Collaborators
-------------

* Pablo Estigarribia (pablodav at gmail)
