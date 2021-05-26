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

- new_image: This is the name (not the location) of the ISO image that you want to install on your BIG-IP. This can be either a HotFix or a major version ISO. This must be accessible on the control node.

- new_image_dir: This is the location of the ISO image that you want to install. This must be accessible on the control node.

- backup_loc: This is the location that UCS backups will be stored in. This location is **ON THE BIG-IP** not the ansible control node.

- backup_pfx: This is a prefix that will be added to the ucs backup. This is useful is you want to use the date, or some other identifier to denote the work being done (i.e. a change number).
