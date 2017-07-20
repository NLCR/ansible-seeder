NLCR.Seeder
=========

Simple role for deploying [Seeder](https://github.com/WebarchivCZ/Seeder) to CentOS server.

Role Variables
--------------

```
---
# Seeder settings
seeder_home: /opt/Seeder
seeder_virtualenv: /opt/virtualenv/seeder
seeder_user: uwsgi
seeder_group: "{{ seeder_user }}"
seeder_logs: /var/log/seeder

# local_settings.py
seeder_secret: sdfvsdfvsdfvsdfv
seeder_debug: True
seeder_template: "{{ seeder_debug }}"
seeder_thumbnail: "{{ seeder_debug }}"
seeder_allowed: "{{ ansible_hostname }}', '{{ ansible_all_ipv4_addresses | join(\"', '\") }}"
seeder_sentry: 'http://7b446e212dcb48b2ab68f44b91d268e8:3baf62247b0145b4a8a2c095486682a8@10.3.0.20:9000/4'

# seeder postgresql
seeder_db_name: seeder
seeder_db_user: seeder
seeder_db_pass: seeder
seeder_db_host: 127.0.0.1
# legacy db
wadmin_db_connected: False
# others
seeder_elastic_host: "localhost:9200"
seeder_elastic_index: haystack
seeder_memcached: "localhost:11211"
seeder_manet_host: "localhost:8891"
wakat_url: 'kat.webarchiv.cz'
...
```
Dependencies
------------

NLCR.elasticsearch, NLCR.postgresql, NLCR.supervisor

and optional: NLCR.wakat, NLCR.nginx, NLCR.mariadb. 

Example Playbook
----------------
```
 - name: Seeder imports old data from legacy WAdmin DB.
   hosts: seeder
   roles:
     - role: NLCR.mariadb
       mariadb:
         name: '{{ wadmin.db }}'
         import: wadmin-2017-02-01.sql.bz2
         user: '{{ wadmin.user }}'
         pass: '{{ wadmin.pass }}'
       tags: wadmin

 - name: Web archiving catalogization assistence as web service
   hosts: seeder
   roles:
     - role: NLCR.wakat
       wa_kat:
         path: /mnt/raid1/git/WA-KAT
         release: v1.1.8
         port: 10001
       tags: wa-kat

 - name: Seeder wants Elasticsearch with analysis_icu.
   hosts: seeder
   roles:
     - role: NLCR.elasticsearch
       elastic:
         analysis_icu: True
       tags: elastic

 - name: Nginx server
   hosts: seeder
   roles:
     - role: NLCR.nginx
       tags: nginx

 - name: PostgreSQL database for Seeder
   hosts: seeder
   roles:
     - role: NLCR.postgresql
       postgresql:
         - user: '{{ seeder_db.user }}'
           pass: '{{ seeder_db.pass }}'
           db: '{{ seeder_db.db }}'
       tags: postgresql

 - name: Supervisor for Seeder
   hosts: seeder
   roles:
     - role: NLCR.supervisor
       tags: supervisord

 - name: Seeder curation tool
   hosts: seeder
   roles:
     - role: NLCR.seeder
       seeder_home: /mnt/raid1/git/Seeder
       seeder_hostname: war.webarchiv.cz
       seeder_allowed: "some domain', 'Some IP"
       seeder_debug: False
       seeder_sentry: '{{ seeder.sentry }}'
       wadmin_db_connected: True
       wa_kat:
         hostname: kat.webarchiv.cz
         port: 10001
       tags: seeder
```
