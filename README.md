# CrostiniTips
## Tips for running linux containers (LXC) on ChromeOS via Crostini

Crostini is a system for running LXC containers on the ChromeOS operating system. Crostini was designed with a focus on insulating the operating system from attacks or exploits coming from the containers. This is great for developers who want a secure, managed, operating system; while also having access to a linux development environment.

Crostini's architecture is complex. There are several layers of virtualization technologies that are nested within each other. As well as communication and control channels that traverse layers. This document will help you understand this architecture so that you can run and troubleshoot LXC containers on ChromeOS. 

## Crostini Architecture Overview
![Crostini Services Diagram](./images/crostini_services.png "crostini_services.png")

This architecture diagram from the [Crostini Developer Guide](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/crostini_developer_guide.md) is important for understanding Crostini's various subsystems and how they interact. It can be read in tandem with [Running Custom Containers Under Chrome OS](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/containers_and_vms.md), where the Overview sections gives a concise description of each of the subsystems.

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
Add appropriate apt repo key
Add repo
apt install cros-guest-tools
https://chromium.googlesource.com/chromiumos/containers/cros-container-guest-tools/+/refs/heads/main/lxd/lxd_setup.sh

## Getting started running custom LXC containers.
In ChromeOS settings, go to Advanced > Developers> Linux development environment and enable "Linux development environment."

In Chrome type `chrome://flags#crostini-multi-container` and enable the feature.

Restart your Chromebook.

Depending on your version of ChromeOS you may now have access to the multi-container UI.

## Troubleshooting
* If Linux Developer Options do not appear or the multiple container UI doesn't appear, try restarting the system. Or try disabling and re-enabling the crostini-multi-container flag, or linux development environment.

* If you are working off of tutorials, check that they are somewhat recent, ChromeOS/Crostini are developing rapidly.


## Remotes for LXC container images
https://us.lxd.images.canonical.com/
https://us.lxd.images.canonical.com/meta/1.0/
https://us.lxd.images.canonical.com/meta/1.0/index-system.1
https://us.lxd.images.canonical.com/meta/1.0/index-user.1
