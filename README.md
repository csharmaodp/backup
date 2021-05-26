# f5-upgrade-ansible

This playbook is an example ansible playbook for upgrading an F5 BIG-IP.

It builds upon the work of https://github.com/twpsyn/f5-upgrade

## The fine print
This playbook has been tested for both major upgrades and hotfixes.
It does perform some basic checks, but it is not exhausitve in what it checks for.

Most importantly - you use it at your own risk :)

## Variables
There are a number of variables that need to be set or passed into the playbook in order for it to function properly.

The ansible v1 modules for F5 BIG-IP need a provider block for authentication (this changes in v2 but for now these require the provider).

```
provider:
  server: "{{ inventory_hostname }}"
  user: **YOUR USERNAME**
  password: **YOUR PASSWORD**
  validate_certs: no

    new_image: "BIGIP-16.0.1.1-0.0.6.iso"

    new_image_dir: "/root/ansible/upgrade"

    backup_loc: "/var/local/ucs"

    backup_pfx: "ref12345"
```
