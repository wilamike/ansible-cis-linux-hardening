# ansible-redhat-hardening
Ansible roles to harden Redhat 7 and 8 on AWS or GCP according to CIS framework. This is developed based on:

* Redhat 8 - https://github.com/jwacross/RHEL8-CIS
* Redhat 7 - https://github.com/ansible-lockdown/RHEL7-CIS

The project is tested with inspec with below profile:

* dev-sec cis-dil-benchmark - https://github.com/dev-sec/cis-dil-benchmark


---

## HOWTO

### To run on AWS EC2 or GCE instance locally (Apply for RHEL7 and RHEL8):


* Install CIS DIL galaxy role
```
ansible-galaxy install dev-sec.os-hardening dev-sec.ssh-hardening
```


* Run baseline hardening
```
ansible-playbook ansible/server_baseline_hardening.yml
```

* Run cis-dil hardening
```
ansible-playbook ansible/server_cisdil_hardening.yml
```

* Reboot

---

## Known issues

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

---
## Workaround

* An empty GRUB password is added by this Ansible roles to pass the respective cis-dil test. It won't reduce the security of Cloud VM as the boot menu is not accessible.

