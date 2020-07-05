---
title: Install Docker on CentOS 8
description: Install Docker on CentOS 8 and configure firewalld.
layout: post
permalink: install-docker-centos-8
image: "img/docker-centos-8/docker-centos-8-install.png"
---
CentOS 8 is already here with major updates. One of them being the replacement of well known `yum` with `dnf` package manager. All these updates make the upgrade from CentOS 7 to CentOS 8 difficult and it's even not supported officially. So, in most cases people end up building new CentOS 8 servers and migrating the apps to new infrastructure, or try and upgrade it manually risking running an unsupported version of the OS. You can find more details on [this forum thread](https://forums.centos.org/viewtopic.php?t=71848#p302574 "CentOS 7 to CentOS 8 upgrade script - CentOS").

While I was organizing another 1337.MD CTF (the 5th one whoop-whoop!), I decided to upgrade whole infrastructure where CTFd platform was running. Before, it was running on CentOS 7 server with docker-ce and docker-compose installed on it. So the platform was running inside a container, which gave me a easier way to upgrade it. Since majority of articles on the web are still referring to CentOS 7 (even the official Docker documentation), I decided to write this post and help other folks possibly.

**NOTE** `For production >= CentOS 8 environments you probably should use podman instead! You will have to convert docker-compose files into kubernetes objects, by using kompose tool`


## Create new user

Let's prepare the system first, before we can install `docker-ce` and `docker-compose`. If you intend to run docker from a less privileged user, which I definitely recommend, then you'll need to create a separate user for that:

```bash
sudo useradd dockeruser
sudo passwd dockeruser
```


## Install docker-ce

Starting with CentOS 8, docker is replaced with the `daemon-less Docker alternative` called `podman`. So we need to add community edition repository:

```bash
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```

Check if `dnf` package manager finds it successfully by running:

```bash
sudo dnf list docker-ce
```

By running this command you should see the latest version of `docker-ce` listed: `19.03.12-3.el7`. However it fails to install because of a broken dependency of `containerd.io` package. Latest version of docker needs `containerd.io >= 1.2.2-3`, which is not available by default on CentOS 8 yet. At this point we have 2 options:

- Manually install latest version of `containerd.io` package and get latest `docker-ce` version;
- Go for `18.09.1-3.el7` version of docker;

There are drawbacks in both approaches. If we take the first approach, then we end up having a package which needs to be managed separately from docker-ce (it is not listed as dependency). The ugly part about the second approach is that there are 2 critical security vulnerabilities in `18.09.1-3.el7`, which have been patched in `19.03.12-3.el7`. These are:

- [CVE-2017-18367](https://nvd.nist.gov/vuln/detail/CVE-2017-18367 "libseccomp-golang 0.9.0 and earlier incorrectly generates BPFs"): libseccomp-golang 0.9.0 and earlier incorrectly generates BPFs;
- [CVE-2019-14271](https://nvd.nist.gov/vuln/detail/CVE-2019-14271 "code injection can occur when the nsswitch facility dynamically loads a library"): In Docker 19.03.x before 19.03.1 linked against the GNU C Library (aka glibc), code injection can occur when the nsswitch facility dynamically loads a library inside a chroot that contains the contents of the container.

### Installing latest docker-ce

Manually install `containerd.io`:

```bash
sudo dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
```

Now we can install the latest `docker-ce`:

```bash
sudo dnf install docker-ce
```

### Installing older docker-ce
DNF package manager, by default, is configured to consider for installation best candidates only. Since latest version of `docker` doest not have all necessary dependencies ported to CentOS 8 yet, we will have to use `--nobest` flag. It tells `dnf` to fallback, in case it fails to solve the dependencies issues, and install the older version without any dependency issues.

```bash
dnf install docker-ce --nobest -y
```


## Firewall configuration

Usually you will get issues with container connectivity and DNS resolution. To solve this, we need to configure `firewalld`.

```bash
firewall-cmd --zone=trusted --change-interface=docker0
firewall-cmd --zone=trusted --add-masquerade --permanent
firewall-cmd --reload
```

## Final thoughts

Not sure if we will get a proper support for `docker` on CentOS 8 in future. I will have to check the `podman` and docker-compose files conversion with `kompose` and post here in future.