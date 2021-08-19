# Upgrading F5 BIG-IP using Ansible

This playbook is an example Ansible playbook for upgrading an F5 BIG-IP.

It builds upon the work of https://github.com/twpsyn/f5-upgrade

## The fine print
This playbook has been tested for both major software releases as well as hotfixes.
It does perform some basic checks, but it is not exhaustive in what it checks for.

Most importantly - you use it at your own risk (and be sure to include more pre- and post-checks when using for production devices) 

## Installation
In order to use the playbook you will need to have the F5 Ansible modules (v1) installed on your control node.
```
ansible-galaxy collection install f5networks.f5_modules
```

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

- **new_image**: This is the name (not the location) of the ISO image that you want to install on your BIG-IP. This can be either a HotFix or a major version ISO. This must be accessible on the control node.

- **new_image_dir**: This is the location of the ISO image that you want to install. This must be accessible on the Ansible control node.

- **backup_loc**: This is the location that UCS backups will be stored in. This location is **ON THE BIG-IP** not the Ansible control node.

- **backup_pfx**: This is a prefix that will be added to the ucs backup. This is useful is you want to use the date, or some other identifier to denote the work being done (i.e. a change window number or internal service request number).

## Assumptions
There are several important assumptions that the playbook makes:

- That the address in your Ansible hosts file is the management address of the device.
- That the user you use has admin priviliges on the BIG-IP.
- That your F5 devices are in a group called "F5" in your ansible hosts file.

## How to run
In order to run the playbook you use the following command:
```
ansible-playbook -i f5-hosts f5-upgrade.yml
```

If you want to override the variables that are passed into the playbook you can do this:
```
ansible-playbook -i f5-hosts -e "new_image_dir=/foo" f5-upgrade.yml
```

If you want to pass in the multi level YAML as part of the provider, you can do this:
```
ansible-playbook -i f5-hosts -e '{ "bigip_provider": { "password": "foo"}  }'
```

For most people the first method will work just fine.

## Caveats
During testing it was found that the BIG-IP needs to have an appropriate amount of resources in order to proess the upgrade. This has to do with the process restjavad on the BIG-IP needing an appropriate amount of memory in order to receive API calls (e.g by provisioning Management to Large). Under the covers, the Ansible modules are making API calls that are handled by this process.

## Sample output
The following represents full sample output from a major version upgrade on a test device (VE).

```
TASK [Wait For Confirmation] ***************************************************************************************************************************
[Wait For Confirmation]
WARNING: THIS IS COMPLETELY UNTESTED AND BY HITTING ENTER YOU ACCEPT WHATEVER HAPPENS TO YOUR LIFE HEREAFTER (either good or bad):
ok: [10.1.1.245]

TASK [Get Software Volume Information] *****************************************************************************************************************
ok: [10.1.1.245]
fatal: [10.1.1.246]: UNREACHABLE! => {"changed": false, "msg": "Failed to connect to the host via ssh: ssh: connect to host 10.1.1.246 port 22: No route to host", "unreachable": true}

TASK [Get Current Version] *****************************************************************************************************************************
ok: [10.1.1.245] => (item={'status': 'complete', 'product': 'BIG-IP', 'name': 'HD1.1', 'base_build': '0.0.31', 'version': '15.1.0', 'build': '0.0.31', 'active': 'yes', 'full_path': 'HD1.1'})

TASK [Set New Version from ISO name] *******************************************************************************************************************
ok: [10.1.1.245]

TASK [Identify Hosts That Require Upgrade] *************************************************************************************************************
ok: [10.1.1.245]

TASK [Identify Hosts That Don't Require Upgrade] *******************************************************************************************************
skipping: [10.1.1.245]

TASK [Identify Hosts need Hotfix] **********************************************************************************************************************
skipping: [10.1.1.245]

TASK [Identify Hosts that DO NOT need Hotfix] **********************************************************************************************************
skipping: [10.1.1.245]

TASK [Check For Only One Boot Location] ****************************************************************************************************************
ok: [10.1.1.245]

TASK [Check First Boot Location] ***********************************************************************************************************************
skipping: [10.1.1.245]

TASK [Check Second Boot Location] **********************************************************************************************************************
skipping: [10.1.1.245]

TASK [Device Version Status] ***************************************************************************************************************************
ok: [10.1.1.245] => {
    "msg": [
        "Current version: 15.1.0",
        "Desired image: BIGIP-16.0.1.1-0.0.6.iso",
        "Upgrade needed: True"
    ]
}

TASK [Print Upgrade Information] ***********************************************************************************************************************
ok: [10.1.1.245] => {
    "msg": [
        "Current version: 15.1.0 booting from HD1.1",
        "New Image 'BIGIP-16.0.1.1-0.0.6.iso' will be uploaded from '/root/ansible/upgrade'",
        "It will be installed to boot location 'HD1.2'"
    ]
}

TASK [Wait For Confirmation] ***************************************************************************************************************************
[Wait For Confirmation]
Press a key to continue...:
ok: [10.1.1.245]

TASK [Save the running configuration of the BIG-IP] ****************************************************************************************************
changed: [10.1.1.245]

TASK [Ensure backup directory exists] ******************************************************************************************************************
ok: [10.1.1.245]

TASK [Get Pre-Upgrade UCS Backup] **********************************************************************************************************************
changed: [10.1.1.245]

TASK [Upload image] ************************************************************************************************************************************
changed: [10.1.1.245]

TASK [Install Image] ***********************************************************************************************************************************
changed: [10.1.1.245]

TASK [Activate image] **********************************************************************************************************************************
ok: [10.1.1.245]

TASK [Pausing execution to give device time to reboot (first time)] ************************************************************************************
Pausing for 120 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [10.1.1.245]

TASK [Pausing execution to give device time to reboot (second time - to give you the illusion of progress)] ********************************************
Pausing for 120 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [10.1.1.245]

TASK [Pausing execution to give device time to reboot (third and final time)] **************************************************************************
Pausing for 60 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [10.1.1.245]

TASK [wait for ssh to come up] *************************************************************************************************************************
ok: [10.1.1.245]

TASK [Wait for device to become active or standby (URI method)] ****************************************************************************************
FAILED - RETRYING: Wait for device to become active or standby (URI method) (20 retries left).
FAILED - RETRYING: Wait for device to become active or standby (URI method) (19 retries left).
FAILED - RETRYING: Wait for device to become active or standby (URI method) (18 retries left).
ok: [10.1.1.245]

TASK [Get Post-Upgrade UCS Backup] *********************************************************************************************************************
changed: [10.1.1.245]

PLAY RECAP *********************************************************************************************************************************************
10.1.1.245                 : ok=21   changed=5    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
```
