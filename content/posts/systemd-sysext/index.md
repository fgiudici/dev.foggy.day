---
title : 'systemd-sysext notes'
date : 2025-02-16
draft : false
summary: 'systemd-sysext activates/deactivates system extension images. [...] (man systemd-sysext)'
cover:
  image : images/folders.png
  hiddenInList: true
---

*systemd-sysext activates/deactivates system extension images. [...]* (man systemd-sysext)

## Overview
Systemd `systemd-sysext` merges *system extension images* (we will refer to them as `extension images`
from here on) into the OS root fs via OverlayFS.

`Extension images` are files and directories provided in one of the following format:
* plain directories
* btrfs subvolumes
* fs images (squasfh, ext4, erofs, …)
* GPT partition images

The only paths that are merged from the folders and files present in the `extension images` are **/usr**
and **/opt**. These two paths become read-only [^1] on the merged fs, also if they were mutable (rw) in 
the base OS fs.

[^1]: Actually, the images can be merged **rw** specifying the `--mutable=`[`no`, `auto`, `yes`, `import`, `ephemeral`, `ephemeral-import`] option

All files in the extension images are made available instantly and atomically.

`Extension images` must be placed in:
* /etc/extensions/
* /run/extensions/
* /var/lib/extensions/

## Basic Usage
```bash
systemd-sysext refresh [--force]
```
This will check the extension image folders, unmerge all the images and freshly merge all the available
ones. The /usr and /opt dir, if present in one of the extension images,  will become read-only (if they
weren't yet).

```bash
systemd-sysext merge [--force]
```
Merges available extensions images. If some extension images are already merged and new extension images are
detected the command fails (requires a `systemd-sysext unmerge` first).

```bash
systemd-sysext unmerge
```
Unmerges all the currently merged extension images.

> [!NOTE]
Systemd provides also the `systemd-sysext.service` unit file to activate the available `extension images`.

## Compatibility Enforcement
Each `extension image` requires a paired extension-release file containing information about its
ompatibility.

The extension-release file must match the image *NAME*: so, an `extension image` called *NAME*.raw requires
an extension-release file `/usr/lib/extension-release.d/extension-release.NAME`.

The extension-release file is compared with the host os-release file (/etc/os-release).

The mandatory fields that have to match are:
* `ID`
* either `SYSEXT_LEVEL` or `VERSION_ID`

otherwise the extension image will not be merged.

You can disable the compatibility check by:
* using `ID=_any` in the extension-release file
* using the `--force` flag with the `systemd-sysext merge` or `refresh` commands

## Systemd units in extension images
If systemd unit files are shipped inside the `extension image`, the service manager needs to be reloaded
in order to detect them. This can be achieved adding the field `EXTENSION_RELOAD_MANAGER=1` in the
extension-release file.

> [!Tip]
Systemd unit files included in `extension images` are not started also if loaded at boot by the
`systemd-sysext service` as the unit files become available too late.
>
>The Flatcar project solves the issue by providing the [`ensure-sysext service`](https://github.com/flatcar/init/blob/flatcar-master/systemd/system/ensure-sysext.service) which triggers systemd to reload and restart
the service files after the `system-sysext service` has merged the available `extension images`.

## systemd-confext
The `systemd-confext` tools is similar to `systemd-sysext` but targets `extension images` for configuration
files (it merges the **/etc** path).

Exec permission is disabled (by default).

`systemd-confexts` extension images should be placed in one of:
* /run/confexts
* /var/lib/confexts
* /usr/lib/confexts
* /usr/local/lib/confexts

## :wrench: Debugging
```bash
systemd-dissect [--list] $NAME.raw
```
Use systemd-dissect to check the `extension image` file properties.

To debug `systemd-sysext` commands, prepend (or export) `SYSTEMD_LOG_LEVEL=debug`.

## :link: External links
* [Lennart Pottering systemd-sysext blogpost](https://0pointer.net/blog/testing-my-system-code-in-usr-without-modifying-usr.html)

* [Flatcar systemd-sysext intro](https://www.flatcar.org/docs/latest/provisioning/sysext/#the-sysext-format)

* [Flatcar systemd-sysext speech at FrOSCon 2022](https://media.ccc.de/v/froscon2022-2775-deploy_software_with_systemd-sysext)

* [Flatcar systemd-sysext examples](https://github.com/flatcar/sysext-bakery)
