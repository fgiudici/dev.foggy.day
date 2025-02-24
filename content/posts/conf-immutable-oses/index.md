---
date: '2025-02-24'
title: 'Configuring Immutable OSes'
summary: 'Immutable OSes usually allows or even require declarative configuration:
          brief overview of the cloud-init, ignition, butane and combustion tools and formats'
cover:
  image: images/cloud-conf.png
  hiddenInList: true
---

Immutable OSes (like
[OpenSUSE MicroOS](https://microos.opensuse.org/),
[Flatcar Container Linux](https://www.flatcar.org/),
[Fedora COREOS](https://fedoraproject.org/coreos/), ... )
allow or even strictly require to be configured by means of declarative configuration.

The most used tools are `cloud-init`, `ignition`, `butane` and `combustion`: let's look a bit more into them.

## Cloud-Init
Is the first declarative configuration format that has been widely used, introduced by Canonical quite some years ago, used also in public cloud deployments.

The syntax is in `yaml` format and the files should always start with `#cloud-config`.

:memo: https://cloud-init.io/  
:floppy_disk: https://github.com/canonical/cloud-init

Sample file:
```yaml
#cloud-config

groups:
  - admingroup: [root,sys]
  - cloud-users

users:
  - default
  - name: foobar
    gecos: Foo B. Bar
    primary_group: foobar
    groups: users
    selinux_user: staff_u
    ssh_import_id:
      - lp:falcojr
      - gh:TheRealFalcon
    passwd: $6$j212wezy$7H/1LT4f9/N3wpgNunhsIqtMj62OKiS3nyNwuizouQc3u7MbYCarYeAHWYPYb2FT.lbioDm2RrkJPb9BZMN1O/
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSL7uWGj8cgWyIOaspgKdVy0cKJ+UTjfv7jBOjG2H/GN8bJVXy72XAvnhM0dUM+CCs8FOf0YlPX+Frvz2hKInrmRhZVwRSL129PasD12MlI3l44u6IwS1o/W86Q+tkQYEljtqDOo0a+cOsaZkvUNzUyEXUwz/lmYa6G4hMKZH4NBj7nbAAF96wsMCoyNwbWryBnDYUr6wMbjRR1J9Pw7Xh7WRC73wy4Va2YuOgbD3V/5ZrFPLbWZW/7TFXVrql04QVbyei4aiFR5n//GvoqwQDNe58LmbzX/xvxyKJYdny2zXmdAhMxbrpFQsfpkJ9E/H5w0yOdSvnWbUoG5xNGoOB csmith@fringe
write_files:
- encoding: b64
  content: CiMgVGhpcyBmaWxlIGNvbnRyb2xzIHRoZSBzdGF0ZSBvZiBTRUxpbnV4...
  owner: root:root
  path: /etc/sysconfig/selinux
  permissions: '0644'
- content: |
    # My new /etc/sysconfig/samba file
    SMBDOPTIONS="-D"
  path: /etc/sysconfig/samba
```

## Ignition
*Ignition is a utility created to manipulate disks during the initramfs.*

`Ignition` has been introduced in CoreOS and replaced the usage of `cloud-init` there.

It is currently used by:
* [Fedora CoreOS](https://fedoraproject.org/coreos/)
* [Red Hat Enterprise Linux CoreOS](https://docs.openshift.com/container-platform/4.17/architecture/architecture-rhcos.html)
* [Flatcar Container Linux](https://www.flatcar.org/)
* [OpenSUSE MicroOS](https://microos.opensuse.org/)
* [SUSE Linux Micro](https://www.suse.com/products/micro/)

The syntax is versioned and in `json` format.

:memo: https://coreos.github.io/ignition/  
:floppy_disk: https://github.com/coreos/ignition

>[!Tip]
There is an online :link:[Ignition & Combustion Config Generator](https://opensuse.github.io/fuel-ignition/)
from OpenSUSE.

Sample file:
```json
{
  "ignition": { "version": "3.0.0" },
  "passwd": {
    "users": [
      {
        "name": "systemUser",
        "passwordHash": "$superSecretPasswordHash.",
        "sshAuthorizedKeys": [
          "ssh-rsa veryLongRSAPublicKey"
        ]
      },
      {
        "name": "jenkins",
        "uid": 1000
      }
    ]
  }
}
```

## Butane
*Butane (formerly the Fedora CoreOS Config Transpiler, FCCT) translates human readable Butane Configs into machine readable Ignition Configs.*

Butane provides a yaml config that can be *transpiled* to [Ignition](#ignition) json config using the `butane` cli tool.

Its only usage is to produce [Ignition](#ignition) configuration files.

The syntax is versioned and in `yaml` format.

:memo: https://coreos.github.io/butane/  
:floppy_disk: https://github.com/coreos/butane

Sample file:
```yaml
variant: fcos
version: 1.1.0
passwd:
  users:
    - name: user1
      ssh_authorized_keys:
        - key1
      home_dir: /home/user1
      no_create_home: true
      groups:
        - wheel
        - plugdev
      shell: /bin/bash
```

## Combustion
*Combustion is a minimal module for dracut, which runs a user provided script on the first boot of a system.*

Combusion is available on OpenSUSE MicroOS and SUSE Micro immutables OSes only.

It allows the execution of custom commands and custom scripts at first boot to provide maximum configuration
flexibility.

The syntax is the one of a shell script.

:memo: https://en.opensuse.org/Portal:MicroOS/Combustion  
:floppy_disk: https://github.com/openSUSE/combustion

>[!Tip]
There is an online :link:[Ignition & Combustion Config Generator](https://opensuse.github.io/fuel-ignition/)
from OpenSUSE.


Sample file:
```sh
#!/bin/bash
# combustion: network
# Redirect output to the console
exec > >(exec tee -a /dev/tty0) 2>&1
# Set a password for root, generate the hash with "openssl passwd -6"
echo 'root:$5$.wn2BZHlEJ5R3B1C$TAHEchlU.h2tvfOpOki54NaHpGYKwdNhjaBuSpDotD7' | chpasswd -e
# Add a public ssh key and enable sshd
mkdir -pm700 /root/.ssh/
cat id_rsa_new.pub >> /root/.ssh/authorized_keys
systemctl enable sshd.service
# Install vim-small
zypper --non-interactive install vim-small
# Leave a marker
echo "Configured with combustion" > /etc/issue.d/combustion
# Close outputs and wait for tee to finish.
exec 1>&- 2>&-; wait;
```


