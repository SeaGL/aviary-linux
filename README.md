# Aviary Linux

# Purpose

This is a purpose-built, atomic/immutable Linux operating system designed to have everything needed to run live broadcasting on-site at [SeaGL](https://seagl.org/). It is intended to be installed on laptops that run broadcasting in each talk room, and is based on [Universal Blue](https://github.com/ublue-os) which is itself based on [Fedora Silverblue](https://fedoraproject.org/atomic-desktops/silverblue/).

# Architecture

This repo uses [BlueBuild](https://blue-build.org/) to build ostree-compatible container images. Commits trigger container builds that are pushed to GitHub Container Registry.

There are _two_ container images in play, for performance reasons. The base image takes a long time to build (both inherently and because it is chunk-optimized after being built, to improve incremental upgrade performance) and includes almost all binary software distributed with Aviary. The final image builds on the base image and includes most configuration data in the system, as well as Aviary-specific scripts. It is not chunk-optimized, but observe that the vast, vast majority of the final image in terms of size has been optimized because these top non-optimized bits are extremely small.

The purpose of this separation is to make it very fast to cut and deploy new releases that update only configuration, or that fix bugs in Aviary-specific code - on the order of a few minutes.

# Prerequisites

Working knowledge in the following topics:

- Containers
  - https://www.youtube.com/watch?v=SnSH8Ht3MIc
  - https://www.mankier.com/5/Containerfile
- rpm-ostree
  - https://coreos.github.io/rpm-ostree/container/
- Fedora Silverblue (and other Fedora Atomic variants)
  - https://docs.fedoraproject.org/en-US/fedora-silverblue/
- GitHub Workflows
  - https://docs.github.com/en/actions/using-workflows

# Installation

This procedure was tested on one of the conference's streaming laptops; you may need to adjust otherwise.

1. Write and boot a Fedora Silverblue (as of October 2024, version 40) installer. On conference laptops, the boot menu key is F12.
2. Select Automatic partitioning and check the checkbox to free space by removing or resizing existing partitions.
3. When prompted, remove **all** partitions, including and especially the EFI System Partition.
4. Run the installer.
5. Reboot.
6. Go through setup.
   1. Connect to WiFi. NOTE that as of Fedora 40, captive portal login (like UW uses) does not work during initial setup. Use a phone hotspot to get through initial setup, then you can switch to UW's WiFi once you're on the desktop.
   2. Leave location services enabled.
   3. Enable Third-Party Repositories and click Next.
   4. Set Full Name to "SeaGL Provisioning".
   5. Accept the default username of `seaglprovisioning`.
   6. Set the password to `password`.
9. Apply firmware updates in GNOME Software, if applicable. **This is very important** as once you've switched to Aviary Linux, you can't apply these anymore due to EFI partition naming shenanigans.
8. In a terminal, run `sudo rpm-ostree rebase ostree-unverified-registry:ghcr.io/seagl/av-linux:latest`. The system should print logs of what it's doing as it works, but you can monitor progress of this step by running `rpm-ostree status` and/or `sudo journalctl -fu rpm-ostreed.service` in a new terminal window.
9. When rpm-ostree rebase finishes (i.e. when `rpm-ostree status` reports `Status: idle`), reboot.
10. In a terminal, rebase to the signed image with `sudo rpm-ostree rebase ostree-image-signed:docker://ghcr.io/seagl/av-linux:latest`. You will have an `age` password prompt which you can either close or ignore (doesn't matter) - just open a new terminal window.
11. When `rpm-ostree status` reports `Status: idle`, reboot.
