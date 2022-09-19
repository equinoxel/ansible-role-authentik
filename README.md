Role Name
=========

This role installs [Authentik](https://goauthentik.io/) via a docker-compose file.

Requirements
------------

Your machine needs to have *docker* and *docker-compose* installed

Role Variables
--------------

This role uses the variables listed below, along with default values (see defaults/main.yml).

### Volumes

Authentik has various components (db, redis, geoIP); a **path-based** volume is defined for each:

```yml
# base path to be used by others as default
authentik_volume_base: "/mnt/authentik"
# media
authentik_volume_config: "{{ authentik_volume_base }}/config"
# media
authentik_volume_media: "{{ authentik_volume_base }}/media"
# certs for https
authentik_volume_certs: "{{ authentik_volume_base }}/certs"
# geoip db location
authentik_volume_geoip: "{{ authentik_volume_base }}/geoip"
# custom templates
authentik_volume_templates: "{{ authentik_volume_base }}/templates"
# db
authentik_volume_db: "{{ authentik_volume_base }}/db"
# redis
authentik_volume_redis: "{{ authentik_volume_base }}/redis"
```

You should define at least `authentik_volume_base` to point to your desired location. You may want specific volumes pointing to different places, in which case you need to override (some of) the above variables.

### Authentik

You can select the docker image version.

```yml
authentik_image_version: "2022.8.2"
```

Authentik uses a secret key, which you **must** set. You can also set the error reporting flag (see Authentik documentation)

```yml
authentik_secret_key: "changeme"
authentik_error_reporting: "false"
```

You should also define the exported authentik ports (ports mapped from docker):

```yml
authentik_port_http: 80
authentik_port_https: 443
```

You also can define a custom location for the GeoIP database:

```yml
# Allow the DB to be located somewhere else
#
authentik_authentik_geoip: "/geoip/GeoLite2-City.mmdb"
```

### PostgreSQL

Authentik depends on PostgreSQL. All parameters (host, port, database, credentials) are defined below and van be changed:

```yml
authentik_db_host: "postgresql"
authentik_db: "authentik"
authentik_db_user: "authentik"
authentik_db_password: "changeme"
authentik_db_port: "5432"
```

You can expose PostgreSQL to the outside world (e.g. for backup) by defining `authentik_db_container_public_port` to a valid port number.

**Note**:Because of a PostgreSQL limitation, only passwords up to 99 chars are supported. See [this link](https://www.postgresql.org/message-id/09512C4F-8CB9-4021-B455-EF4C4F0D55A0@amazon.com) for details.

### SMTP configuration

Authentik needs a SMTP relay to send various emails. Please change the following parameters:

```yml
authentik_email_host: "localhost"
authentik_email_port: "25"
# Optionally authenticate (don't add quotation marks to you password)
authentik_email_username:
authentik_email_password:
# Use StartTLS
authentik_email_use_tls: "false"
# Use SSL
authentik_email_use_ssl: "false"
authentik_email_timeout: "10"
# Email address authentik will send from, should have a correct @domain
authentik_email_from: "authentik@localhost"
```


### GeoIP

By default, the role installs a GeoIP container, where you require credentials. You can disable this via `authentik_geoip_container`.

```yml
# geoip credentials
authentik_geoip_container: true
geoip_account_id:
geoip_license_key: 
geoip_update_edition_ids: "GeoLite2-City GeoLite2-Country"
geoip_update_frequency: "8"
```

Dependencies
------------

This role needs `community.docker.docker_compose`, which should be available by default.

Example Playbook
----------------

A minimal configuration should have the following variables defined:

1. `authentik_volume_base`.
2. `authentik_port_*`,
3. `authentik_db_password`,
4. `authentik_secret_key` and
5. `authentik_geoip_container` as *false*.

```yml
- hosts: servers
  vars:
    #############################
    # Authentik configuration   #
    #############################
    authentik_error_reporting: "false"
    authentik_volume_base: "~/authentik"
    authentik_port_http: "30001"
    authentik_port_https: "30002"
    # In secrets: 
    # authentik_db_password
    # authentik_secret_key

    #############################
    # GeoIP configuration       #
    #############################
    authentik_geoip_container: false

  roles:
    - 'laurivan.authentik'
```

# Helpers

Once you have installed Authentik, you will need to log in to the system. To do this, you can create a recovery key with the following steps:

1. Log in on the magine where you have Authentik running
2. go to `~/authentik` of the user who ran the ansible role
3. run `docker-compose run --rm server create_recovery_key 10 akadmin`

This will end up with a path along the lines:

> /recovery/use-token/*ReallyLongToken*/

Which you can append to your authentik's server address.


License
-------

MIT

Author Information
------------------

This role was created in 2022 by [Laur Ivan](https://www.laurivan.com)

