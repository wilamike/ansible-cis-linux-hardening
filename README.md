# ansible-cis-linux-hardening
Ansible roles to harden Redhat 7/8 and Ubuntu 18/20 on AWS or GCP according to CIS framework. This is developed based on:

* Redhat 8 - https://github.com/jwacross/RHEL8-CIS
* Redhat 7 - https://github.com/ansible-lockdown/RHEL7-CIS
* Ubuntu 18.04 - https://github.com/florianutz/Ubuntu1804-CIS

The project is tested with inspec with below profile:

* dev-sec cis-dil-benchmark - https://github.com/dev-sec/cis-dil-benchmark


---

## HOWTO

### To run on AWS EC2 or GCE instance locally:

* Tune parameters in below two files for your OS respectively to meet your requirements.
```
ansible/roles/cisdil-redhat-7/defaults/main.yml
ansible/roles/cisdil-redhat-8/defaults/main.yml
ansible/roles/cisdil-debian-18/defaults/main.yml
ansible/roles/cisdil-debian-20/defaults/main.yml
```

* Install galaxy roles
```
ansible-galaxy install -r ansible/requirements.yml
```

* Run baseline hardening
```
ansible-playbook ansible/server_baseline_hardening.yml
```

* Run cis hardening
```
ansible-playbook ansible/server_cisdil_hardening.yml
```

* Reboot

### To customize parameters

* Edit the default settings in below files
```
ansible/roles/cisdil-<OS type>-<OS release>/defaults/main.yml
```

---

## Known issues

You may find below errors when using [Chef Inspec](https://doc.chef.io/inspec/) to verify the setup. 

* The daemon tcpd/tcp_wrapper is depreciated on Redhat and CentOS 8. Hence the respective cis-dil test fails with below message:
```
  ×  cis-dil-benchmark-3.4.1: Ensure TCP Wrappers is installed (2 failed)
     ×  System Package tcpd is expected to be installed
     ×  System Package tcp_wrappers is expected to be installed
```

* The output syntax of sestatus (SELinus Status) has changed since Redhat/CentOS 7. The test in cis-dil is using the Redhat 6 output for parsing and hence fails with below message:
```
 ×  cis-dil-benchmark-1.6.1.3: Ensure SELinux policy is configured (1 failed)
     ×  Command: `sestatus` stdout is expected to match /Policy from config file:\s+(targeted|mls)/
```
Below is the respective output from Redhat 7 and onward.
```
     Loaded policy name:             targeted
```

* GCE VM uses vFAT for boot partition format, and disabling FAT will corrupt the VM reboot. Hence the related fix and check are disabled in Ansible role. Below error messages are returned by cis-dil.
```
  ×  cis-dil-benchmark-1.1.1.8: Ensure mounting of FAT filesystems is disabled (2 failed)
     ×  Kernel Module vfat is expected not to be loaded
     ×  Kernel Module vfat is expected to be disabled
```

* The tests in cis-dil mandate /home, /var, /var/log and /var/log/audit to be mounted to dedicated partition. If you don't care about it as the nature that Cloud VM is ephemeral, you may skip the steps to move them to the new volumes. Below messages will be returned as a result:
```
  ×  cis-dil-benchmark-1.1.6: Ensure separate partition exists for /var
     ×  Mount /var is expected to be mounted
  ×  cis-dil-benchmark-1.1.11: Ensure separate partition exists for /var/log
     ×  Mount /var/log is expected to be mounted
  ×  cis-dil-benchmark-1.1.12: Ensure separate partition exists for /var/log/audit
     ×  Mount /var/log/audit is expected to be mounted
  ×  cis-dil-benchmark-1.1.13: Ensure separate partition exists for /home
     ×  Mount /home is expected to be mounted
  ×  cis-dil-benchmark-1.1.14: Ensure nodev option set on /home partition
     ×  Mount /home options is expected to include "nodev"
```

* The playbook directly generates GRUB password to user.cfg. Meanwhile inspec detects the syntax in config file grub.conf and hence the related test will fail. Below errors are expected.

```
  ×  cis-dil-benchmark-1.4.2: Ensure bootloader password is set (14 failed)
     ×  File /boot/grub/grub.conf content is expected to match /^set superusers/
     ×  File /boot/grub/grub.conf content is expected to match /^password/
     ×  File /boot/grub/grub.cfg content is expected to match /^set superusers/
     ×  File /boot/grub/grub.cfg content is expected to match /^password/
     ×  File /boot/grub/menu.lst content is expected to match /^set superusers/
     ×  File /boot/grub/menu.lst content is expected to match /^password/
     ×  File /boot/boot/grub/grub.conf content is expected to match /^set superusers/
     ×  File /boot/boot/grub/grub.conf content is expected to match /^password/
     ×  File /boot/boot/grub/grub.cfg content is expected to match /^set superusers/
     ×  File /boot/boot/grub/grub.cfg content is expected to match /^password/
     ×  File /boot/boot/grub/menu.lst content is expected to match /^set superusers/
     ×  File /boot/boot/grub/menu.lst content is expected to match /^password/
     ×  File /boot/grub2/grub.cfg content is expected to match /^set superusers/
     ×  File /boot/grub2/grub.cfg content is expected to match /^password/
```

* The below files are set with file permission 0000 instead of 0600 in order to pass Tenable scan test. Below errors are expected.

```
  × cis-dil-benchmark-6.1.3: Ensure permissions on /etc/shadow are configured (2 failed)
    ✔ File /etc/shadow is expected to exist
    × File /etc/shadow is expected to be readable by owner
      expected File /etc/shadow to be readable by owner
    × File /etc/shadow is expected to be writable by owner
      expected File /etc/shadow to be writable by owner

  × cis-dil-benchmark-6.1.5: Ensure permissions on /etc/gshadow are configured (2 failed)
    ✔ File /etc/gshadow is expected to exist
    × File /etc/gshadow is expected to be readable by owner
      expected File /etc/gshadow to be readable by owner
    × File /etc/gshadow is expected to be writable by owner
      expected File /etc/gshadow to be writable by owner

  × cis-dil-benchmark-6.1.7: Ensure permissions on /etc/shadow- are configured (2 failed)
    ✔ File /etc/shadow- is expected to exist
    × File /etc/shadow- is expected to be readable by owner
      expected File /etc/shadow- to be readable by owner
    × File /etc/shadow- is expected to be writable by owner
      expected File /etc/shadow- to be writable by owner

  × cis-dil-benchmark-6.1.9: Ensure permissions on /etc/gshadow- are configured (2 failed)
    ✔ File /etc/gshadow- is expected to exist
    × File /etc/gshadow- is expected to be readable by owner
      expected File /etc/gshadow- to be readable by owner
    × File /etc/gshadow- is expected to be writable by owner
```

* The minimum days between password changes is changed to 3 in order to pass Tenable scan test. Below errors are expected.

```
  ×  cis-dil-benchmark-5.4.1.2: Ensure minimum days between password changes is 7 or more
     ×  login.defs PASS_MIN_DAYS is expected to cmp >= 7
```
---
## Workaround


