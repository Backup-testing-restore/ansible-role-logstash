Ansible Role: Logstash
=========

An extendible Ansible Role that installs Logstash on Debian/Ubuntu. Have default customizing pipelines, predefined configuration for typical usages and recommendations of official documentation.

Requirements
------------

Requires at least Java 8 and connection with Elastic Search 7.x or greater.

Role Variables
--------------

### Required to set or necessary variables (see `vars/main.yml`):

— Node name in ELK cluster.

    node_name: "logstash-node"

— Login and password for system privilege user in Elastic Search.

    elasticsearch_logstash_system_username: ""
    elasticsearch_logstash_system_password: ""

— Login and password for user for writing logs in Elastic Search.

> `username` is optional if `elasticsearch_logstash_auto_create_elastic_user` is `true`.

> Unused if `logstash_ssl_authorities_force_peer` is `true`.
> 
    elasticsearch_logstash_user_name: ""
    elasticsearch_logstash_user_password: ""

— Local paths to the SSL certificate and key files, which will be copied into the `logstash_ssl_dest_dir`.

See [Generating a self-signed certificate](#generating-a-self-signed-certificate) for information about generating and using self-signed certs with Logstash and Filebeat.

> Unused if `logstash_ssl_enable` is `false`.

    logstash_ssl_key_file: ""
    logstash_ssl_certificate_file: ""

— Optional, path to certificate authority, which used for trust connection with Filebeat and for Elastic `xpack.monitoring`.

> Unused if `logstash_ssl_enable` is `false`.

    logstash_ssl_authorities_certificate_file: ""

— Additional parameter for auto creation of role and user in Elastic Search by http requests. Useful for simple scenarios and testing environment.

    elasticsearch_logstash_auto_create_elastic_user: false


### Available variables with default values (see `defaults/main.yml`):

— The major version of Logstash to install.

    logstash_version: '7.x'

— The specific package to be installed.

    logstash_package: logstash

— The directory inside which Logstash is installed.
    
    logstash_dir: /usr/share/logstash

— The hosts where Logstash should provide logs to Elasticsearch.

    logstash_elasticsearch_hosts:
    - https://127.0.0.1:9200
 
—The port over which Logstash will listen for **ELK Beats**.

    logstash_listen_port_beats: 5044

— Operating system user and group for whom file access is granted.

    logstash_service_os_user: root
    logstash_service_os_group: root

— Set this to `false` if you don't want logstash to run on system startup.

    logstash_enabled_on_boot: true

— Set this to `false` if you don't want logstash create service in systemd.
    
    logstash_setup_os_service: true

— A list of Logstash plugins that should be installed.

    logstash_install_plugins:
    - logstash-input-beats
    - logstash-filter-multiline

— Provide predefined settings with correct simple configuration and enabled `xpack.monitoring`. 

    logstash_setup_default_settings: true

— Additional settings with default settings you can add it by `logstash_default_settings_extra_options`.

    logstash_default_settings_extra_options: ""

— Flag for activating Logstash automatic configuration reload.

    logstash_use_config_auto_reload: true

— Set this to `false` if you don't want to add the default config files shipped with this role (configs for Beats, Elastic Search with auth, and useful filters). 

    logstash_setup_default_config: true

— You can add your own configuration files to be placed in `/etc/logstash/conf.d`.

    logstash_custom_configs_list: []

— Whether configuration for local syslog file (defined as `logstash_local_syslog_path`) should be added to logstash. Set this to `false` if you are monitoring the local syslog differently, or if you don't care about the local syslog file. 
    
    logstash_monitor_local_syslog: true
    logstash_local_syslog_path: /var/log/syslog

— Setup security connection with ssl certificates with Filebeat and Elastic Search.

    logstash_ssl_enable: true

— The directory inside which Logstash holding ssl key and certificates files.

    logstash_ssl_dest_dir: /etc/logstash/certs

— Change verify mode of Elastic Search connection to validate connection by TSL certificate by use CA key and certificate. 
> Note: See more details in [Generating a self-signed certificate](#generating-a-self-signed-certificate). 

    logstash_ssl_authorities_enable: true

— Used for disable Basic Auth to use only TSL certificates for Elastic Search connection.

    logstash_ssl_authorities_force_peer: false

— Parameters of request for auto creating role in Elastic Search.

    elasticsearch_logstash_auto_create_role_name: logstash_writer
    elasticsearch_logstash_role_settings: "..."



## Generating a Self-signed certificate

For utmost security, you should use your own valid certificate and keyfile, and update the `logstash_ssl_*` variables in your playbook to use your certificate.

To generate a self-signed certificate/key pair, you can use use the command:
```bash
    $ openssl req -x509 -batch -nodes -days 3650 -newkey rsa:2048 -keyout logstash.key -out logstash.crt -subj '/CN=example.com'
```
> Note that Filebeat and Logstash may not work correctly with self-signed certificates unless you also have the full chain of trust (including the Certificate Authority for your self-signed cert) added on your server. 
> 
> See: https://github.com/elastic/logstash/issues/4926#issuecomment-203936891

Newer versions of Filebeat and Logstash also require a pkcs8-formatted private key, which can be generated by converting the key generated earlier, e.g.:
```bash
    openssl pkcs8 -in logstash.key -topk8 -nocrypt -out logstash.p8
```

If you want to use trust mechanism by TSL certificate with Filebeats you need to set path to CA certificate in `logstash_ssl_authorities_certificate_file` and generate your user certificate with CA key and certificate.  

> Note: See more details in [Official Filebeat documentation](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-ssl-logstash.html).

It also use TSL certificate in CA authority with Elastic Search `xpack.monitoring`.

Example of certificate generation with CA key+certificate:
```bash
    openssl x509 -days 3650 -req -sha512 -in client.csr -CAserial ca.serial -CA ca.crt -CAkey ca.key -out logstash.crt
```

Dependencies
------------

None.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

Without SSL setup and with auto created role and user for Elastic Search:
```yaml
    - hosts: servers
      roles:
        - geerlingguy.java
        - infra.elasticsearch
        - role: infra.logstash
          vars:
            elasticsearch_logstash_system_username: "elastic"
            elasticsearch_logstash_system_password: "Qwerty12345"
            elasticsearch_logstash_user_password: "Qwerty12345"
            elasticsearch_logstash_auto_create_elastic_user: true
            logstash_ssl_enable: false
            logstash_elasticsearch_hosts:
            - http://127.0.0.1:9200
```

Default setup with SSL certificates:
```yaml
    - hosts: servers
      roles:
        - geerlingguy.java
        - infra.elasticsearch
        - role: infra.logstash
          vars:
            elasticsearch_logstash_system_username: "elastic"
            elasticsearch_logstash_system_password: "Qwerty12345"
            elasticsearch_logstash_user_name: "logstash-internal"
            elasticsearch_logstash_user_password: "Qwerty12345"
            logstash_ssl_key_file: "/path/to/logstash.key"
            logstash_ssl_certificate_file: "/path/to/logstash.crt"
            logstash_ssl_authorities_certificate_file: "/path/to/ca.crt"
```
License
-------

MIT / BSD

