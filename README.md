# CrostiniTips
## Tips for running linux containers (LXC) on ChromeOS via Crostini

[Crostini](https://chromeos.dev/en/linux) is a system for running Linux in [LXC](https://linuxcontainers.org/lxc/introduction/) containers on the ChromeOS operating system. Crostini was designed with a focus on insulating ChromeOS from attacks or exploits coming from inside the containers. This is great for developers who want a secure, managed, operating system; while also having access to a linux development environment.

LXC containers provides the user a full operating environment, including a command-line shell; and optionally, a graphical user interface. It is comparable to a virtual machine, except that LXC containers are less demanding on resources (CPU, memory) because they share the system kernel amongst multiple containers. Rather than needing separate kernels per virtual machine. LXC containers differ from Docker containers, in that Docker is mainly used to virtualize single applications, like a web-server, rather than a whole interactive operating environment.   

Crostini's architecture is complex. There are several layers of virtualization technologies nested within each other; as well as control channels that traverse layers. The ChromeOS setup instructions are sufficient for basic operation. However, understanding the architecture and terminology is helpful for tracking down solutions in the event that something doesn't breaks. And understanding how the system is designed is important when making use of the more advanced capabilities. 

## Quickstart Guide
Crostini runs well without any major configuration. If you just want to get started running Linux on your ChromeOS device, the [instructions](https://chromeos.dev/en/linux/setup) from Google are good. ChromeOS devices from 2019 onwards generally support Crostini. If your device is older, you can check if it is [supported](https://sites.google.com/a/chromium.org/dev/chromium-os/chrome-os-systems-supporting-linux). If your ChromeOS device is controlled by an educational organization, the feature may have been disabled.

Before following the Google install directions, one small improvement is to go into the Chrome browser and type `chrome://flags#crostini-multi-container` in the address bar, and enable this feature. As of Chrome version 98, on the Beta channel, this enables some additional options in the user interface to manage multiple linux instances.

## ChromeOS vs ChromiumOS and Open Source Software
ChromiumOS is Open Source Software, the project was [founded by Google](https://blog.chromium.org/2009/12/whats-difference-between-chromium-os.html). Software contributors, including Google employees in their professional capacities, contribute to ChromiumOS. Google then pulls from the ChromiumOS source tree to build their commercial ChromeOS products. The Chromium web browser operates on an identical model, being the underlying source code for Google's Chrome browser. For simplicity, we will refer to ChromeOS and Chrome browser.

## Crostini Architecture Overview

![Crostini Services Diagram](./images/crostini_services.png "crostini_services.png")

This architecture diagram from the [Crostini Developer Guide](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/crostini_developer_guide.md) is helpful for understanding Crostini's various subsystems and how they interact. It can be read in tandem with [Running Custom Containers Under Chrome OS](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/containers_and_vms.md), where the Overview sections gives a concise description of each of the subsystems. The following description is intended as a reference you can come back to, and may make more sense after getting some experience with running LXC linux containers.

On this chart, CrOS is ChromeOS, the host's operating system (light blue background). Termina VM (pink background) is a special, read-only, virtual machine instance that runs the LXD software that manages the LXC containers. Note it is the name of a virtual machine instance, rather than a service. In traditional terms it can be referred to as a "guest operating system."

Two important pieces, not pictured, are [crosh](https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/crosh) and [crossvm](https://google.github.io/crosvm/). Crosh is a limited command-line shell that can interact with the ChromeOS operating system. It is used for launching virtual machines. Crossvm is a "virtual machine monitor". In more traditional terms it can be thought of as a [type-2](https://en.wikipedia.org/wiki/Hypervisor#Classification) hypervisor.

Debian Container (light yellow background) is an LXC container. The default LXC container in named Penguin. User-launched containers would be represented here on the diagram. The nesting hierarchy of the various virtualization environments can be confusing. This tree shows the relationships.    

```
ChromeOS
  ├── Chrome (Browser process)
  └── CrossVM (Hypervisor)
      └── Termina VM (Virtual Machine instance)
          └── LXD (Daemon)
              ├── Penguin LXC (Container)
              └── LXC 2 (Optional user-specified container)
```

[Garcon](https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/vm_tools/garcon/) and [Sommelier](https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/vm_tools/sommelier/) (yellow) are daemons that run inside LXC containers. They are installed with the [cros-continer-guest-tools](https://chromium.googlesource.com/chromiumos/containers/cros-container-guest-tools/+/refs/heads/main) package. They communicate directly with the operating system, crossing the isolation boundary of the virtual machine.

## cros-container-guest-tools
The [cros-continer-guest-tools](https://chromium.googlesource.com/chromiumos/containers/cros-container-guest-tools/+/refs/heads/main) enable communication between a LXC container and the ChromeOS operating system, traversing the CrosVM virtual machine boundary. These tools are installed by attaching a virtual disk inside the LXC container and mounting it at `/opt/google/cros-containers`. [lxc_setup.sh](https://chromium.googlesource.com/chromiumos/containers/cros-container-guest-tools/+/refs/heads/main/lxd/lxd_setup.sh) is run when the container is first created, which installs the `cros-guest-tools`. The guest tools are started in the lxc container via systemd unite files in `/etc/systemd/users/`.    

### Installing cros-container-guest-tools on alternative containers
Add appropriate apt repo key - https://www.google.com/linuxrepositories/
Add repo
apt install cros-guest-tools
https://chromium.googlesource.com/chromiumos/containers/cros-container-guest-tools/+/refs/heads/main/lxd/lxd_setup.sh


For Redhat:
https://centos.pkgs.org/8/epel-x86_64/cros-guest-tools-1.0-0.39.20200806git19eab9e.el8.noarch.rpm.html

Arch Linux (reported broken):
https://aur.archlinux.org/packages/cros-container-guest-tools-git/

## Getting started running custom LXC containers.
In ChromeOS settings, go to Advanced > Developers> Linux development environment and enable "Linux development environment."

In Chrome type `chrome://flags#crostini-multi-container` and enable the feature.

Restart your ChromeOS device.

Depending on your version of ChromeOS you may now have access to the "Manage Extra Containers" option under Advanced > Linux Development Environment.

![Manage Extra Containers](./images/ManageExtraContainers.png "ManageExtraContainers.png")

At this point you have a pretty powerful setup. You can start/stop create/delete default LXC containers from this UI. From the desktop you can right-click (Alt-click) on the Terminal application, to choose which LXC container to connect to.

## Installing Ubuntu
Ubuntu is based on Debian, just like the default Penguin container. As such, the `cros-guest-tools` .deb installer mostly works. Sommelier doesn't install, so Wayland/GUI apps don't work, but you can get the integration with the default Terminal app working for a usable command-line experience.

You can use Advanced > Developers > Linux Development Enviornment > Manage extra containers > Create dialog to get started.

![Ubuntu Install](./images/UbuntuInstall.png "UbuntuInstall.png")

Verify that the container started, by opening Termina and executing `lxc list`, where you should see your container running. Execute `lxc exec <container name> -- /bin/bash`, which should give you a root prompt inside your container.

In that container:
```
apt update
apt upgrade -y
apt install -y wget gpg
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | apt-key add -
echo "deb https://storage.googleapis.com/cros-packages/98 bullseye main" > /etc/apt/sources.list.d/cros.list"
apt update
apt install -y cros-guest-tools
```
At some point this install will fail. Possibly on the `cros-ui-config` package. This appears related to the Chromium OS Sommelier package which provides the Wayland service for graphical dekstop apps.

Shutdown the container with `shutdown -P now`

In the `termina` virtual machine, you can verify the container is stopped with `lxc list`. Exit out of `termina` and in the `crosh` shell execute `vmc container termina <container name>`

On my system this immediately dropped me into a shell as user `ubuntu` on the lxc container. SystemD had started the garcon daemon, and right-clicking (alt-clicking) on the Terminal app allowed me to open a shell into the container. The container was also able to be stopped and deleted via the Manage Extra Containers dialog. The container was available across reboots, and just needed to be re-started via the Terminal app. 

Further research can be done into de-coupling the gui and sound bits from the `cros-guest-tools` .deb meta-package, and only installing those guest extensions needed for a minimal Ubuntu install.

Possible workarounds to GUI issue: 
https://github.com/quack1-1/scripts
https://github.com/pitastrudl/wekanwiki/blob/master/Chromebook.md#5-install-crostini-packages
https://github.com/LukeShortCloud/rootpages/blob/main/src/linux_distributions/chromium_os.rst

## Installing Red Hat Linux distributions.




## Advanced LXC container operations

lxc list (defaults to containers)

lxc config show <container name>

lxc profile, lxc profile show <default>

lxc remote list

If you launch an image that you don't have locally, it will first download it. To avoid this you can cp it locally first. be sure to --copy-aliases

Adding a new remote:


lxc info --resources (information about running server)
for more info, append --debug

And number of profiles can be applied to a container. So you could have one that handles networking, another that handles mounts, etc.


## Networking
Under settings you can do port forwarding.

## Files Access

## USB

## ChromeOS internals
For advanced troubleshooting, low-level development, or curiosity; one might wish to understand how ChromeOS works behind the scense. Especially considering that there appears to be some "magic" configuration of LXC containers.

It is possible to put a ChromeOS device into Developer Mode, by holding down ESC+Refresh while powering up the device. You will be prompted with warnings that continuing will wipe user data off the device. Once you complete the ChromeOS device setup, you can enter CRoSH shell (Ctrl+Alt+T) and then type `shell` to log into a non-virtualized, "bare-metal" shell on the device. You should see a prompt of `chronos@localhost`. The `chronos` user has sudo privileges.

ChromeOS uses [Upstart](https://www.chromium.org/chromium-os/chromiumos-design-docs/boot-design/) as it's initialization (init) system. System configuration is stored in `/etc/init/`, and `/sbin/initctl` can be used to quest the state of services. By default neither `/sbin` or `/usr/sbin` are in `$PATH`, although you will find useful executables there.

The command `ps aux --forest` is useful in seeing how the running system is organized. You can see user, process, and child-process information.

The control virtual machine, named termina is loaded from `/run/imageloader/termina-dlc/package/root/`

## Troubleshooting
* If Linux Developer Options do not appear or the multiple container UI doesn't appear, try restarting the system. Or try disabling and re-enabling the crostini-multi-container flag, or linux development environment.

* If you are working off of tutorials, check that they are somewhat recent, ChromeOS/Crostini are developing rapidly.


___
Working below this line.


https://chromium.googlesource.com/chromiumos/docs/+/HEAD/costini_developer_guide.md
https://chromium.googlesource.com/chromiumos/docs/+/HEAD/containers_and_vms.md
https://chromium.googlesource.com/chromium/src/+/main/chrome/browser/ash/crostini
https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/vm_tools/vsh/

https://chromium.googlesource.com/chromiumos/docs/+/HEAD/dbus_in_chrome.md
https://www.chromium.org/developers/mus-ash/ - discontinued 2019

https://www.chromium.org/chromium-os/chromiumos-design-docs/boot-design/

https://chromium.googlesource.com/chromiumos/platform2/

https://chromium.googlesource.com/chromiumos/docs/+/HEAD/security/chromeos_security_whitepaper.md - Graphic clarifying OS Userspace is here.

https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/crosh/src/base/vmc.rs

https://www.chromium.org/chromium-os/

https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/vm_tools/docs/init.md

https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/vm_tools/init/vm_concierge.conf - nsenter details

https://chromium.googlesource.com/chromiumos/containers/cros-container-guest-tools/+/refs/heads/main/lxd/lxd_setup.sh#35 -Confirms that Tremplin writes /etc/apt/sources.d/cros.list (Possible also writes sytemd unit files then?)

Original place where cros-containers repo was found.
https://www.reddit.com/r/Crostini/wiki/howto/install-behind-a-proxy
https://storage.googleapis.com/cros-containers

https://www.aboutchromebooks.com/news/chrome-os-98-adds-management-of-multiple-chromebook-linux-containers/



https://www.reddit.com/r/Crostini/comments/s31fwc/multiple_crostini_containers_can_now_run/

https://blog.simos.info/a-closer-look-at-chrome-os-using-lxd-to-run-linux-gui-apps-project-crostini/#The%20Chrome%20OS%20deb%20package%20repository

https://github.com/edeloya/ChromeOS-Terminal-LXC-LXD
https://chromium.googlesource.com/chromiumos/overlays/board-overlays/+/HEAD/project-termina/
https://medium.com/@tcij1013/lxc-lxd-cheetsheet-effb5389922d

https://wiki.archlinux.org/title/Chrome_OS_devices/Crostini
https://wiki.debian.org/LXC

https://blog.merzlabs.com/posts/crostini-now-usable/
https://blog.merzlabs.com/posts/yubikey-crostini/


https://stgraber.org/2016/03/11/lxd-2-0-blog-post-series-012/

https://linuxcontainers.org/lxd/docs/master/authentication/

## Source code for reference
https://chromium.googlesource.com/chromiumos/platform2/
https://chromium.googlesource.com/chromiumos/platform/crosvm/
https://github.com/google/crosvm

https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/crosh

https://chromium.googlesource.com/chromiumos/containers/cros-container-guest-tools/


https://chromium.googlesource.com/chromiumos/platform/tremplin/
https://chromium.googlesource.com/chromiumos/overlays/board-overlays/+/main/project-termina/

https://github.com/lxc

## Remotes for LXC container images

https://us.lxd.images.canonical.com/
https://us.lxd.images.canonical.com/meta/1.0/
https://us.lxd.images.canonical.com/meta/1.0/index-system.1
https://us.lxd.images.canonical.com/meta/1.0/index-user.1
