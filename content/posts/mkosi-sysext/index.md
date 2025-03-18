---
date: '2025-03-18'
title: 'Building System Extension Images with mkosi'
summary: "mkosi (make operating system image) is a tool to generates OS images using
          package managers and repositories of many linux distributions. Let's see how
          mkosi can help building System Extension Images."
cover:
  image: images/files-and-folders.png
  hiddenInList: true
---

mkosi (**m**a**k**e **o**perating **s**ystem **i**mage) is a powerful tool to generate OS fs images.

It allows to pick a distribution and build the OS fs image starting from the available packages there.

When targeting an opensuse distro, it retrieves the files and folders packaged in the distro rpm by
acting as a `zypper --buildroot` wrapper.

For an overall overview and introduction of the tool see the
[A re-introduction to mkosi -- A Tool for Generating OS Images](https://0pointer.net/blog/a-re-introduction-to-mkosi-a-tool-for-generating-os-images.html)
blogpost from Daan De Meyer.

Here we will assume basic knowledge of the tool and will focus on building System Extension Images.

## Format=sysext Output
mkosi allows to directly specify a *sysext* format output in the config file.

The *sysext* image must be built on top of a *base* image.

As an example, let's build a system extension image from the *helm* package from OpenSUSE Tumbleweed:

*mkosi.conf*
```ini
[Distribution]
Distribution=opensuse
Release=tumbleweed
[Output]
OutputDirectory=mkosi.output
[Build]
CacheDirectory=mkosi.cache
[Validation]
Verity=no
```

*mkosi.images/base/mkosi.conf*
```ini
[Output]
Format=directory
[Content]
Packages=
 systemd
 udev
```

*mkosi.images/helm/mkosi.conf*
```ini
[Config]
Dependencies=base
[Output]
Format=sysext
Overlay=yes
[Validation]
Verity=no
[Content]
BaseTrees=%O/base
Packages=helm
```

running `mkosi` in the base dir triggers the creation of the base image (we picked up a a plain
directory format there as we don't really need an image for it) and the creation of the `helm`
system extension image.

The generated artifacts are made available in the *mkosi.output* directory.

>[!Note]
The *sysext* format is an automated and *opinionated way* to build a system extension disk image.
Inspecting the *mkosi.output/helm.raw* image
(see [*debugging* in *systemd-sysext notes*]({{< ref "posts/systemd-sysext/#wrench-debugging" >}}))
you can see that mkosi created:
>* a GTP partition containing a single partition formatted with an *erofs* file system
>* a `/usr/lib/extension-release.d/extension-release.helm` file matching ID and ARCHITECTURE from the
`/etc/os-release` of the base image

## Custom System Extension Image 
If the desired system extension image requires a different file system than the *erofs* one or a
customized partitioning is needed, a *normal* disk format should be specified in the mkosi configuration
file and the required configurations (the extension-release file and the systemd-repart configuration)
should be provided explicitly.
>[!tip]
The partitioning configuration must be specified in the `mkosi.repart` subfolder.
Additional files and folders to be copied in the final image should be put into the `mkosi.extra` folder.

Let's provide an example configuration for a system extension image for our helm package from OpenSUSE
Tumbleweed: this time we want it packed in a *squashfs* file system and we want the extesion-release
file to carry a `ID=_any` value to make the image compatible with multiple linux systems:

*mkosi.conf* (stays the same)
```ini
Distribution]
Distribution=opensuse
Release=tumbleweed
[Output]
OutputDirectory=mkosi.output
[Build]
CacheDirectory=mkosi.cache
[Validation]
Verity=no
```
*mkosi.images/base/mkosi.conf* (stays the same too)
```ini
[Output]
Format=directory
[Content]
Packages=
 systemd
 udev
```
*mkosi.images/helm/mkosi.conf*
```ini
[Config]
Dependencies=base
[Output]
Format=disk
Overlay=yes
[Validation]
Verity=no
[Content]
BaseTrees=%O/base
Packages=helm
```
*mkosi.images/helm/mkosi.extra/usr/lib/extension-release.d/extension-release.helm* (new)
```ini
ID=_any
ARCHITECTURE=x86-64
```
*mkosi.images/helm/mkosi.repart/root.conf* (new)
```ini
[Partition]
Type=root
Format=squashfs
CopyFiles=/opt/
CopyFiles=/usr/
Minimize=best
```

Note that we changed the *Format* to *disk* in the mkosi configuration file for the helm image,
so we had also to specify ourselves the *extension-release.helm* file required for an extension image.

## :link: External links
:page_facing_up: [Daan De Meyer's mkosi blogpost](https://0pointer.net/blog/a-re-introduction-to-mkosi-a-tool-for-generating-os-images.html)

:clapper: [Daan De Meyer's mkosi talk at All Systems Go! conference](https://www.youtube.com/watch?v=6EelcbjbUa8)

:floppy_disk: [mkosi project](https://github.com/systemd/mkosi)