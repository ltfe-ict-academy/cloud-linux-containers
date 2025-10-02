# Installing and Running LXC

Sources:
- ✅ [What's LXC?](https://linuxcontainers.org/lxc/introduction/)
- ✅ [Github LXC](https://github.com/lxc/lxc)
- ✅ [Install LXC and LXC UI on Ubuntu 22.04|20.04|18.04|16.04](https://computingforgeeks.com/how-to-install-lxc-lxc-ui-on-ubuntu/?expand_article=1)
- ✅ [Containers - LXC](https://ubuntu.com/server/docs/containers-lxc)
- ✅ [How to Manage Linux Containers using LXC](https://www.geeksforgeeks.org/how-to-manage-linux-containers-using-lxc/)
- ✅ [LXC: Linux container tools](https://developer.ibm.com/tutorials/l-lxc-containers/)


## Introduction
LXC is a userspace interface for the Linux kernel containment features. Through a powerful API and simple tools, it lets Linux users easily create and manage system or application containers.

Current LXC uses the following kernel features to contain processes:
- Kernel namespaces (ipc, uts, mount, pid, network and user)
- Apparmor and SELinux profiles
- Seccomp policies
- Chroots (using pivot_root)
- Kernel capabilities
- CGroups (control groups)

LXC containers are often considered as something in the middle between a [chroot](https://securityqueens.co.uk/im-in-chroot-jail-get-me-out-of-here/) and a full fledged virtual machine. The goal of LXC is to create **an environment as close as possible to a standard Linux installation but without the need for a separate kernel**.

> LXC is free software, most of the code is released under the terms of the GNU LGPLv2.1+ license

LXC is currently made of a few separate components:
- The liblxc library
- Several language bindings for the API (python3, lua, go, ruby, haskell)
- A set of standard tools to control the containers
- Distribution container templates


## Releases
LXC is supported by all modern GNU/Linux distributions, and there should already be an LXC package available from the standard package repositories for your distro.

- [Info about releases](https://linuxcontainers.org/lxc/news/)

LXC 6.0, 5.0 and 4.0 are long term support releases:
- LXC 6.0 will be supported until June 1st 2029 (current: LXC 6.0.5)
- LXC 5.0 will be supported until June 1st 2027 (current: LXC 5.0.3)
- LXC 4.0 will be supported until June 1st 2025 (current: LXC 4.0.12)


## Installation
[Requirements](https://linuxcontainers.org/lxc/getting-started/#requirements):
- One of glibc, musl libc, uclib or bionic as your C library
- Linux kernel >= 3.8 (for all features to be available)

In most cases, you'll find recent versions of LXC available for your Linux distribution. Either directly in the distribution's package repository or through some backport channel.

> For your first LXC experience, we recommend you use a recent supported release, such as a recent bugfix release of LXC 4, 5 or 6.

Ubuntu is also one of the few (if not only) Linux distributions to come by default with everything that's needed for safe, unprivileged LXC containers.

On such an Ubuntu system, installing LXC is as simple as:
```bash
sudo apt update
sudo apt upgrade # (optional but recommended)
sudo apt install lxc
```

This will pull in the required and recommended dependencies, as well as set up a network bridge for containers to use.

Your system will then have all the LXC commands available, all its templates as well as the python3 binding should you want to script LXC.

Check the installed version:
- `lxc-info --version`

Here’s the state of LXC (classic Linux Containers) on recent Ubuntu LTS releases:

| Ubuntu release        | LXC version in official repos (current “updates”)  | Notes on upgrading to newer LXC                                                                                                                                                                       |
| --------------------- | -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **20.04 LTS (Focal)** | **4.0.12** (`1:4.0.12-0ubuntu1~20.04.1`)           | That’s the latest you’ll get via `apt` on 20.04. Note: 20.04’s standard support ended **31 May 2025** (now ESM). To get LXC 5/6 you must either upgrade Ubuntu or build from source. |
| **22.04 LTS (Jammy)** | **5.0.0** (package `1:5.0.0~git2209-…-0ubuntu1.1`) | 5.0.x is the latest via `apt` on 22.04. LXC 6 isn’t in Jammy’s official repos. Upgrade Ubuntu or build from source for 6.x.                            |
| **24.04 LTS (Noble)** | **5.0.3** (package `1:5.0.3-2ubuntu7.2`)           | Noble also tracks LXC 5.0.x. No LXC 6 in the official Noble repos. Upgrade OS or build from source for 6.x.  |

> If you want LXC 6.x on 20/22/24 without changing Ubuntu, you’ll need to build from source (Meson) per upstream instructions; otherwise, upgrade the OS to a release that ships LXC 6.0.x by default.

After installing LXC, the following commands will be available in the host system:
```
lxc-attach         lxc-create      lxc-snapshot
lxc-autostart      lxc-destroy     lxc-start
lxc-cgroup         lxc-device      lxc-start-ephemeral
lxc-checkconfig    lxc-execute     lxc-stop
lxc-checkpoint     lxc-freeze      lxc-top
lxc-clone          lxcfs           lxc-unfreeze
lxc-config         lxc-info        lxc-unshare
lxc-console        lxc-ls          lxc-usernsexec
lxc-copy           lxc-monitor     lxc-wait
```

Each of the preceding commands has its own [dedicated manual (man) page](https://linuxcontainers.org/lxc/manpages/), which provides a handy reference for the usage of, available options for, and additional information about the command.

For LXC userspace tools to work properly in the host operating system, you must **ensure that all the kernel features required for LXC support are enabled** in the running host kernel. This can be verified using `lxc-checkconfig`.

> Everything listed in the `lxc-checkconfig` command output should have the status enabled (beside the Cgroup v1 options); otherwise, try restarting the system.

<!-- If using Ubuntu 22.04 or later the following option are missing:
```
Cgroup v1 systemd controller: missing
Cgroup v1 freezer controller: missing
Cgroup ns_cgroup: required
```

Edit `/etc/default/grub` and ensure that `GRUB_CMDLINE_LINUX` contains:
- `GRUB_CMDLINE_LINUX="systemd.unified_cgroup_hierarchy=false"`
- Then run `sudo update-grub` and reboot the system to apply the changes.
- `sudo reboot`
- Then run `lxc-checkconfig` again to verify that all the required kernel features are enabled. -->


## LXC Default Configuration
`/etc/lxc/default.conf` is the default configuration file for LXC installed using the standard Ubuntu packages. This configuration file supplies the default configuration for all containers created on the host system.
- `cat /etc/lxc/default.conf`

> For privileged use, they are found under `/etc/lxc`, while for unprivileged use they are under `~/.config/lxc`.

Add the option for LXC to manage NAT automatically (If lxc-net is active and `LXC_NAT="true"`, it will create/maintain the `MASQUERADE` rule for `lxcbr0` so you don't need your manual iptables lines.):
- Check if lxc-net is running: `sudo systemctl status lxc-net`
- Add `LXC_NAT="true"` to file: `sudo nano /etc/default/lxc-net`
- Restart and enable: `sudo systemctl enable --now lxc-net`

The networking will be set up as a virtual Ethernet connection type—that is, veth from the network bridge lxcbr0 for each container that will get created.

The installation will also configure a default container network. The name of the bridge is `lxcbr0`:
- `ip addr | grep lxc`

LXC supports **user namespaces, and this is the recommended way to secure LXC containers.** User namespaces are configured by assigning user ID (UID) and group ID (GID) ranges for existing users, where an existing system user except root will be mapped to these UID/GID ranges for the users within the LXC container.

Based on user namespaces, LXC containers can be classified into two types:
- Privileged containers
- Unprivileged containers


## Privileged LXC containers
**Privileged** - This is when you run lxc commands as root user.
  - The old-style containers, they're not safe at all and should only be used in environments where unprivileged containers aren't available and where you would trust your container's user with root access to the host.
  - As privileged containers are considered unsafe, new container escape exploits won't be worthy of quick fix.

Privileged containers are **defined as any container where the container uid 0 is mapped to the host's uid 0.** In such containers, protection of the host and prevention of escape is entirely done through Mandatory Access Control (apparmor, selinux), seccomp filters, dropping of capabilities and namespaces.

Those technologies combined will typically prevent any accidental damage of the host, where damage is defined as things like reconfiguring host hardware, reconfiguring the host kernel or accessing the host filesystem.

**LXC upstream's position is that those containers aren't and cannot be root-safe.**

> They are still valuable in an environment where you are running trusted workloads or where no untrusted task is running as root in the container.

We are aware of a number of exploits which will let you escape such containers and get full root privileges on the host. 

The only way to restrict that access is by using the methods previously described, such as seccomp, SELinux, and AppArmor. But writing a policy that applies the desired security that is required can be complicated.

Some of those exploits can be trivially blocked and so we do update our different policies once made aware of them. Some others aren't blockable as they would require blocking so many core features that the average container would become completely unusable.

Privileged containers are **started by the host’s root user**; once the container starts, the **container’s root user is mapped to the host’s root user which has UID 0.** This is the default when an LXC container is created in most of the distros, where there is no default security policy applied.

Create, run and attach to a privileged container:
```bash
# Create a container with the given OS template and options.
sudo lxc-create -t download \
  -n my-first-debian -- \
  --dist debian \
  --release bookworm \
  --arch amd64

# Start running the container that was just created.
sudo lxc-start -d -n my-first-debian
# List all the containers in the host system.
sudo lxc-ls --fancy
# Obtain detailed container information
sudo lxc-info -n my-first-debian
# Get a default shell session inside the container.
sudo lxc-attach -n my-first-debian
# Run ID and exit
id
ps axfwwu
exit
# Stop the container.
sudo lxc-stop -n my-first-debian
# Permanently delete the container. Removes the container, including its rootfs.
sudo lxc-destroy -n my-first-debian
```


## Building a LXC container
We can create our first container using a template. The lxc-download file, like the rest of the templates in the templates directory, is a script written in bash:
- `ls -la /usr/share/lxc/templates/`
- [Image server for Incus and LXC](https://images.linuxcontainers.org/). In LXC, this image server can be used by selecting the `lxc-download` template. 

List all available images from download template:
- `/usr/share/lxc/templates/lxc-download -l`

Let's start by building a container using the lxc-download template, which will ask for the distribution, release, and architecture, then use the appropriate template to create the filesystem and configuration for us:
- `sudo lxc-create -t download -n my-first-ctr`
- Choose the following options:
  - Distribution: ubuntu
  - Release: jammy
  - Architecture: amd64
- Let's list all containers: `sudo lxc-ls -f`
- Our container is currently not running; let's start it in the background and increase the log level to DEBUG: `sudo lxc-start -n my-first-ctr -d -l DEBUG`
- Let's list all containers: `sudo lxc-ls -f`
- To obtain more information about the container run the following: `sudo lxc-info -n my-first-ctr`
- The new container is now connected to the host bridge `lxcbr0`:
  - `brctl show`
  - `ip a s lxcbr0`
  - `ip a s <container-veth-name>`

Using the download template and not specifying any network settings, the container obtains its IP address from a dnsmasq server that runs on a private network, `10.0.3.0/24` in this case. The host allows the container to connect to the rest of the network and the Internet using NAT rules in iptables:
  - `sudo iptables -L -n -t nat`

Other containers connected to the bridge will have access to each other and to the host, as long as they are all connected to the same bridge and are not tagged with different VLAN IDs.

Let's run attach with the container, list all processes, and network interfaces, and check connectivity:
  - `sudo lxc-attach -n my-first-ctr`
  - `ps axfw`
  - `ip a s`
  - `ping -c 3 google.com`
  - `exit`

> Notice how the hostname changed on the terminal once we attached to the container. This is an example of how LXC uses the UTS namespaces.

Let's examine the directory that was created after building the container: 
- `sudo ls -la /var/lib/lxc/my-first-ctr`
- `sudo ls -la /var/lib/lxc/my-first-ctr/rootfs`

The rootfs directory looks like a regular Linux filesystem. You can manipulate the container directly by making changes to the files there.
- `sudo touch /var/lib/lxc/my-first-ctr/rootfs/test-file`
- `sudo lxc-attach -n my-first-ctr`
- `ls -al`
- `exit`

To stop the container, run the following command:
- `sudo lxc-stop -n my-first-ctr`
- `sudo lxc-destroy -n my-first-ctr`


## Unprivileged LXC containers
Unprivileged containers map the container’s root (uid 0) to an unprivileged UID on the host (via user namespaces). This means root inside the container is not root on the host, greatly reducing risk if something goes wrong. Using unprivileged containers is the recommended default for most setups.

- Unprivileged containers are safer than privileged ones because uid 0 in the container maps to a non-root host UID (from your /etc/subuid range). 
- LXC also applies AppArmor/Seccomp/capabilities by default on Ubuntu as extra hardening.

**Requirements:**
- Linux with user namespaces enabled (check: `lxc-checkconfig | grep "User namespace"`).
- The uidmap helpers: `sudo apt install uidmap`.
- Subordinate ID ranges in `/etc/subuid` and `/etc/subgid` for your user (Ubuntu usually sets 65,536 IDs per new user by default).
- Optional but common: a bridge (e.g., lxcbr0) and NAT so containers can reach the internet.

**Helper configs you'll see:**
- `/etc/lxc/lxc-usernet` - allows a user to create veths and attach them to a bridge.
- `newuidmap` / `newgidmap` - set up the UID/GID maps for unprivileged containers.

**Limitations in unprivileged containers:**
- Mounting many filesystem types is restricted.
- Creating device nodes is restricted.
- You can't act on UIDs/GIDs outside your mapped ranges.
- Because of these, **prefer the download template (prebuilt images known to work unprivileged)**.


### Creating unprivileged containers as a user
- ✅ [Ubuntu 18.04 and unprivileged LXC](https://www.kubos.cz/2018/07/20/ubuntu-unprivileged-lxc)

[Follow the steps](https://linuxcontainers.org/lxc/getting-started/#creating-unprivileged-containers-as-a-user) to setup the host:

1. Prerequisites:
```bash
sudo sysctl -w kernel.unprivileged_userns_clone=1
sudo apt update && sudo apt install -y lxc uidmap acl
```
2. Ensure subuid/subgid exist (Ubuntu normally does this at user creation): 
    - `grep "^$(id -un):" /etc/subuid /etc/subgid`

3. By default, your user isn't allowed to create any network device on the host. Allow your user to create veths on lxcbr0 (adjust bridge if different):
    - `echo "$(id -un) veth lxcbr0 10" | sudo tee -a /etc/lxc/lxc-usernet` (This means that "your-username" is allowed to create up to 10 veth devices connected to the lxcbr0 bridge.)
    - `cat /etc/lxc/lxc-usernet`

4. User default config with correct idmap:
```bash
mkdir -p ~/.config/lxc
echo "lxc.include = /etc/lxc/default.conf" > ~/.config/lxc/default.conf

MS_UID=$(awk -F: -v U="$(id -un)" '$1==U{print $2}' /etc/subuid)   # start
CT_UIDS=$(awk -F: -v U="$(id -un)" '$1==U{print $3}' /etc/subuid)  # count
MS_GID=$(awk -F: -v U="$(id -un)" '$1==U{print $2}' /etc/subgid)
CT_GIDS=$(awk -F: -v U="$(id -un)" '$1==U{print $3}' /etc/subgid)

{
  echo "lxc.idmap = u 0 ${MS_UID} ${CT_UIDS}"
  echo "lxc.idmap = g 0 ${MS_GID} ${CT_GIDS}"
} >> ~/.config/lxc/default.conf
```

5. Check the configuration file: `cat ~/.config/lxc/default.conf`

6. If your container storage lives under your home grant traverse (x) so the mapped container root can reach the path. Use *your* map start:
```bash
sudo setfacl -m u:${MS_UID}:x "$HOME"
sudo setfacl -m u:${MS_UID}:x "$HOME/.local"
sudo setfacl -m u:${MS_UID}:x "$HOME/.local/share"
```

7. Create & manage a container:
```bash
# Create a container with the given OS template and options.
systemd-run --user --scope -p "Delegate=yes" -- lxc-create -t download -n my-first-ubuntu -- --dist ubuntu --release jammy --arch amd64

# Start running the container that was just created.
systemd-run --user --scope -p "Delegate=yes" -- lxc-start -d -n my-first-ubuntu

# List all the containers in the host system.
lxc-ls --fancy

# Get a default shell session inside the container.
systemd-run --user --scope -p "Delegate=yes" -- lxc-attach -n my-first-ubuntu

# Run ID and exit
id
ps axfwwu
exit

# Stop the container.
systemd-run --user --scope -p "Delegate=yes" -- lxc-stop -n my-first-ubuntu

# Permanently delete the container.
systemd-run --user --scope -p "Delegate=yes" -- lxc-destroy -n my-first-ubuntu
```

To run unprivileged containers as an unprivileged user, the **user must be allocated an empty delegated cgroup** (this is required because of the leaf-node and delegation model of cgroup2, not because of liblxc).

It is not possible to simply start a container from a shell as a user and automatically delegate a cgroup. Therefore, you need to wrap each call to any of the lxc-* commands in a systemd-run command.
