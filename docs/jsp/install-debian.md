---
id: install-debian
title: Installing on Debian
---

# Installing on Debian and Ubuntu

## Overview

This guide covers RabbitMQ installation on Debian, Ubuntu and distributions based on one of them.

RabbitMQ is included in standard Debian and Ubuntu repositories. However, the versions included are
usually months or even years behind [latest RabbitMQ releases](/changelog.html),
and thus are [out of support](/versions.html).

RabbitMQ release artifacts include a Debian package. Team RabbitMQ also maintains our own [apt repositories](#apt).

Main topics covered in this guide are

 * [Ways of installing](#installation-methods) the latest RabbitMQ version on Debian and Ubuntu
 * [Supported Ubuntu and Debian distributions](#supported-debian-distributions)
 * How to [install RabbitMQ using apt](#apt) from [Bintray](#apt-bintray) or [Package Cloud](#apt-packagecloud), including a [quick start snippet](#apt-bintray-quick-start)
 * How to [install a recent supported Erlang version using apt](#erlang-repositories)
 * [Version Pinning](#apt-pinning) of apt packages
 * [Privilege requirements](#sudo-requirements)
 * [Direct download](#manual-installation) from GitHub
 * How to [manage the service](#managing-service)
 * How to [inspect node and service logs](#server-logs)

and more.


## How to Install Latest RabbitMQ on Debian and Ubuntu

There are two ways to install the most recent version of RabbitMQ on Debian and Ubuntu:

 * Using an [apt repository on Package Cloud or Bintray](#apt) (this option is highly recommended)
 * Downloading the package and [installing it manually](#manual-installation) with `dpkg -i`.
   This option will require manual installation of all package dependencies.

This guide covers both options. In both cases, a supported version of Erlang has to be installed.
On Debian and Ubuntu, the easiest way to do that is [via apt](#erlang-repositories).


## Supported Distributions

Below is a list Debian-based distributions supported by recent RabbitMQ releases:

 * Ubuntu 14.04 through 19.04
 * Debian Stretch (9), Buster (10) and Sid ("unstable")

The package may work on other Debian-based distributions
if [dependencies](#manual-installation) are satisfied (e.g. using the Wheezy backports repository)
but their testing and support is done on a best effort basis.

## Where to Get Recent Erlang Version on Debian and Ubuntu

RabbitMQ needs Erlang/OTP to run. Erlang/OTP packages in
standard Debian and Ubuntu repositories can be significantly out of date
and not [supported by modern RabbitMQ versions](/which-erlang.html).

Most recent Erlang/OTP release series are available from a number of alternative
apt repositories:

<table>
 <thead>
   <tr>
     <td><strong>Erlang Release Series</strong></td>
     <td><strong>Repositories that provide it</strong></td>
     <td><strong>Notes</strong></td>
   </tr>
 </thead>
 <tbody>
   <tr>
     <td>22.x</td>
     <td>
       <ul>
         <li><a href="#apt-bintray-erlang">Debian packages of Erlang</a> by Team RabbitMQ</li>
         <li><a href="https://packages.erlang-solutions.com/erlang/#tabs-debian">Erlang Solutions</a></li>
       </ul>
     </td>
     <td>
       <strong>Supported <a href="https://groups.google.com/forum/#!topic/rabbitmq-users/vcRLhpUdg_o">starting with 3.7.15</a></strong>. See <a href="/which-erlang.html">Erlang compatibility guide</a>.
     </td>
   </tr>
   <tr>
     <td>21.3.x</td>
     <td>
       <ul>
         <li><a href="#apt-bintray-erlang">Debian packages of Erlang</a> by Team RabbitMQ</li>
         <li><a href="https://packages.erlang-solutions.com/erlang/#tabs-debian">Erlang Solutions</a></li>
       </ul>
     </td>
     <td>
       <strong>Supported <a href="https://groups.google.com/forum/#!msg/rabbitmq-users/KbOldePfgYw/cjYzldEJAQAJ">starting with 3.7.7</a></strong>. See <a href="/which-erlang.html">Erlang compatibility guide</a>.
     </td>
   </tr>
   <tr>
     <td>20.3.x</td>
     <td>
       <ul>
         <li><a href="#apt-bintray-erlang">Debian packages of Erlang</a> by Team RabbitMQ</li>
         <li><a href="https://packages.erlang-solutions.com/erlang/#tabs-debian">Erlang Solutions</a></li>
       </ul>
     </td>
     <td>
       <strong>Supported <a href="https://groups.google.com/forum/#!topic/rabbitmq-users/_imbAavBYjY">starting with 3.6.11</a></strong>. See <a href="/which-erlang.html">Erlang compatibility guide</a>.
     </td>
   </tr>
 </tbody>
</table>

This guide will focus on the first option.


## Install Erlang from an Apt Repostory on Bintray

Standard Debian and Ubuntu repositories tend to provide outdated versions of Erlang/OTP. Team RabbitMQ maintains
an apt repository that includes [packages of modern Erlang/OTP releases](https://bintray.com/rabbitmq-erlang/debian/erlang/) for
a number of commonly used Debian and Ubuntu distributions:

 * Ubuntu 18.04 (Bionic)
 * Ubuntu 16.04 (Xenial)
 * Debian Stretch and Buster

The repo provides most recent patch releases in the following Erlang series:

 * 22.x
 * 21.x
 * 20.3.x
 * 19.3.x
 * master (23.x)
 * R16B03 (16.x)

In order to use the repository, it is necessary to

 * Add (import) repository signing key. `apt` will verify package signatures during installation.
 * Add a source list file for the repository
 * Update package metadata
 * Install Erlang packages required by RabbitMQ

### Add Repository Signing Key

In order to use the repository, add [RabbitMQ signing key](/signatures.html) to `apt-key`.
This will instruct apt to trust packages signed by that key. This can be done using
a key server or via direct key download. Direct download is recommended as SKS key server
are prone to overload:

```sh
curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -
```

Using a key server:

```sh
sudo apt-key adv \
  --keyserver "hkps://keys.openpgp.org" \
  --recv-keys "0x0A9AF2115F4687BD29803A206B73A36E6026DFCA"
```

See the [guide on signatures](/signatures.html) to learn more.

### Enable apt HTTPS Transport

In order for apt to be able to download RabbitMQ and Erlang packages from Bintray, the `apt-transport-https` package must be installed:

```sh
sudo apt-get install apt-transport-https
```

### Add a Source List File

As with all 3rd party Apt (Debian) repositories, a file describing the repository
must be placed under the `/etc/apt/sources.list.d/` directory.
`/etc/apt/sources.list.d/bintray.erlang.list` is the recommended location.

The file should have a source (repository) definition line that uses the following
pattern:

```sh
# This repository provides Erlang packages produced by the RabbitMQ team
# See below for supported distribution and component values
deb http://dl.bintray.com/rabbitmq-erlang/debian $distribution $component
```

The next couple of sections discuss what distribution and component values
are supported.

#### Distribution

In order to set up an apt repository that provides the correct package, a few
decisions have to be made. One is determining the distribution name. It typically matches
the Debian or Ubuntu release used but only a handful of distributions are
supported (indexed) by the Erlang Debian packages maintained by Team RabbitMQ:

 * `bionic` for Ubuntu 18.04
 * `xenial` for Ubuntu 16.04
 * `buster` for Debian Buster
 * `stretch` for Debian Stretch

However, not all distributions are covered (indexed) on Bintray.
But there are good news: since the package indexed for these distributions is identical,
any reasonably recent distribution name would suffice
in practice. For example, users of Debian Buster, Debian Sid, Ubuntu Disco and Ubuntu Eoan
can use both `stretch` and `bionic` for distribution name.

Below is a table of OS release and distribution names that should be used on Bintray.

| Release        | Distribution Name to Use on Bintray |
|----------------|-----------|
| Ubuntu 18.04   | `bionic`  |
| Ubuntu 19.04   | `bionic`  |
| Ubuntu 19.10   | `bionic`  |
| Ubuntu 16.04   | `xenial`  |
| Debian Buster  | `buster`  |
| Debian Stretch | `stretch` |
| Debian Sid     | `buster`  |


#### Erlang/OTP Version

Another is what Erlang/OTP release version should be provisioned. It is possible to track
a specific series (e.g. `21.x`) or install the most recent version available. The choice
determines what Debian repository `component` will be configured.

Consider the following repository file at `/etc/apt/sources.list.d/bintray.erlang.list`:

```ini
## Installs the latest 21.x version available in the repository.
## Please see the distribution name table above.
deb http://dl.bintray.com/rabbitmq-erlang/debian bionic erlang-21.x
```

It configures apt to install the most recent Erlang `21.x` version available in the
repository and use packages for Ubuntu 18.04 (Bionic). More recent Ubuntu releases
should also use this distribution name.

For Debian Buster the file would look like this:

```ini
## Installs the latest 21.x version available in the repository.
## Please see the distribution name table above.
deb http://dl.bintray.com/rabbitmq-erlang/debian buster erlang-21.x
```

For Debian Stretch:

```ini
## Installs the latest 21.x version available in the repository.
## Please see the distribution name table above.
deb http://dl.bintray.com/rabbitmq-erlang/debian stretch erlang-21.x
```

To use the most recent `20.x` patch release available, switch the component
to `erlang-20.x`:

```ini
## Installs the latest 20.x version available in the repository.
## Please see the distribution name table above.
deb http://dl.bintray.com/rabbitmq-erlang/debian bionic erlang-20.x
```

`erlang-21.x`, `erlang-19.x`, and `erlang-16.x` are the components for Erlang 21.x,
19.x and R16B03, respectively.

The `erlang` component installs the most recent version available:

```ini
## Installs the latest version available in the repository.
## Consider using version pinning.
## Please see the distribution name table above.
deb http://dl.bintray.com/rabbitmq-erlang/debian bionic erlang
```

That version may or may not be supported by RabbitMQ, so [package version pinning](#apt-pinning) is highly recommended.

### Install Erlang Packages

After updating the list of `apt` sources it is necessary to run `apt-get update`:

```sh
sudo apt-get update -y
```

Then packages can be installed just like with the standard Debian repositories:

```sh
# This is recommended. Metapackages such as erlang and erlang-nox must only be used
# with apt version pinning. They do not pin their dependency versions.
sudo apt-get install -y erlang-base \
                        erlang-asn1 erlang-crypto erlang-eldap erlang-ftp erlang-inets \
                        erlang-mnesia erlang-os-mon erlang-parsetools erlang-public-key \
                        erlang-runtime-tools erlang-snmp erlang-ssl \
                        erlang-syntax-tools erlang-tftp erlang-tools erlang-xmerl
```

### Package Version and Repository Pinning

When the same package (e.g. `erlang-base`) is available from multiple apt repositories operators need
to have a way to indicate what repository should be preferred. It may also be desired to restrict Erlang version to avoid undesired upgrades.
[apt package pinning](https://wiki.debian.org/AptPreferences) feature can be used to address both problems.

Package pinning is configured with a file placed under the `/etc/apt/preferences.d/` directory, e.g. `/etc/apt/preferences.d/erlang`.
After updating apt preferences it is necessary to run `apt-get update`:

```sh
sudo apt-get update -y
```

The following preference file example will configure `apt` to install `erlang-*` packages from Bintray
and not standard Debian or Ubuntu repository:

```ini
# /etc/apt/preferences.d/erlang
Package: erlang*
Pin: release o=Bintray
Pin-Priority: 1000
```

This apt preference configuration is recommended when the `erlang` [repository component is used](#erlang-source-list-file).

Effective package pinning policy can be verified with

```sh
sudo apt-cache policy
```

The following preference file example will pin all `erlang-*` packages to `22.2.1`
(assuming [package epoch](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-Version) for the package is 1):

```ini
# /etc/apt/preferences.d/erlang
Package: erlang*
Pin: version 1:22.2.1-1
Pin-Priority: 1000
```

The following preference file example will pin `rabbitmq-server` package to to `3.8.2`
(assuming [package epoch](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-Version) for the package is 1):

```ini
# /etc/apt/preferences.d/rabbitmq
Package: rabbitmq-server
Pin: version 1:3.8.2
Pin-Priority: 1000
```


In the example below, the `esl-erlang` package is pinned to to to `22.2.1`
(assuming [package epoch](https://www.debian.org/doc/debian-policy/ch-controlfields.html#s-f-Version) for the package is 1):

```ini
# /etc/apt/preferences.d/erlang
Package: esl-erlang
Pin: version 1:22.2.1
Pin-Priority: 1000
```


## Using RabbitMQ Apt Repositories

RabbitMQ packages can be installed from apt (Debian) repositories on [Package Cloud](https://packagecloud.io/rabbitmq/rabbitmq-server)
or [Bintray](https://bintray.com/rabbitmq/debian/rabbitmq-server). Both repositories provide packages
for most recent RabbitMQ releases.

PackageCloud provides a more opinionated and automated way of repository setup. With Bintray
the experience is closer to setting up any other 3rd party apt repository.

### Using RabbitMQ Apt Repository on PackageCloud

PackageCloud is a package hosting service. Team RabbitMQ maintains an [apt repository on PackageCloud](https://packagecloud.io/rabbitmq/rabbitmq-server).

A quick way to install uses a [Package Cloud-provided script](https://packagecloud.io/rabbitmq/rabbitmq-server/install#bash-deb).

There are more installation options available:

 * Using PackageCloud Chef cookbook
 * Using PackageCloud Puppet module
 * Manually

[PackageCloud RabbitMQ repository instructions](https://packagecloud.io/rabbitmq/rabbitmq-server/install) lists all available options.

Package Cloud signs distributed packages using their own GPG keys.

```sh
# import PackageCloud signing key
wget -O - "https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey" | sudo apt-key add -
```

After importing both keys please follow the [Package Cloud repository setup instructions](https://packagecloud.io/rabbitmq/rabbitmq-server/install).

### Using RabbitMQ Apt Repository on Bintray

Bintray is a package distribution service. Team RabbitMQ maintains an [apt repository on Bintray](https://bintray.com/rabbitmq/debian/rabbitmq-server).
When using the repository on Bintray it is recommended that Erlang/OTP is also [installed from Bintray](#installing-erlang-package).

In order to use a 3rd apt repository, it is necessary to

 * [Add (import) repository signing key](#erlang-apt-repo-signing-key). `apt` will verify package signatures during installation.
 * Add a repository file
 * Update package metadata
 * Install the Erlang package

#### Quick Start Example

Below is shell snippet that performs those steps. They are documented in more detail below.

```sh
#!/bin/sh

## If sudo is not available on the system,
## uncomment the line below to install it
# apt-get install -y sudo

sudo apt-get update -y

## Install prerequisites
sudo apt-get install curl gnupg -y

## Install RabbitMQ signing key
curl -fsSL https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc | sudo apt-key add -

## Install apt HTTPS transport
sudo apt-get install apt-transport-https

## Add Bintray repositories that provision latest RabbitMQ and Erlang 21.x releases
sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list &lt;&lt;EOF
## Installs the latest Erlang 22.x release.
## Change component to "erlang-21.x" to install the latest 21.x version.
## "bionic" as distribution name should work for any later Ubuntu or Debian release.
## See the release to distribution mapping table in RabbitMQ doc guides to learn more.
deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang
deb https://dl.bintray.com/rabbitmq/debian bionic main
EOF

## Update package indices
sudo apt-get update -y

## Install rabbitmq-server and its dependencies
sudo apt-get install rabbitmq-server -y --fix-missing
```

#### Add Signing Key

See [Add Repository Signing Key](#erlang-apt-repo-signing-key) above.

#### Enable apt HTTPS Transport

See [Install apt HTTPS transport](#erlang-apt-https-transport) above.

#### Add a Source List File

As with all 3rd party apt repositories, a file describing the RabbitMQ and Erlang package repositories
must be placed under the `/etc/apt/sources.list.d/` directory.
`/etc/apt/sources.list.d/bintray.rabbitmq.list` is the recommended location.

The file should have a source (repository) definition line that uses the following
pattern:

```ini
# Source repository definition example.
# See below for supported distribution and component values

# Use this line to install the latest Erlang 21.3.x package available
deb https://dl.bintray.com/rabbitmq-erlang/debian $distribution erlang-21.x

# Or use this line to install the latest Erlang 22.x package available
# deb https://dl.bintray.com/rabbitmq-erlang/debian $distribution erlang

# This repository provides RabbitMQ packages
deb https://dl.bintray.com/rabbitmq/debian $distribution main
```

The next couple of sections discusses what distribution and component values
are supported.

#### Distribution

In order to set up an apt repository that provides the correct package, a few
decisions have to be made. One is determining the distribution name. It typically matches
the Debian or Ubuntu release used:

 * `bionic` for Ubuntu 18.04
 * `xenial` for Ubuntu 16.04
 * `buster` for Debian Buster
 * `stretch` for Debian Stretch

However, not all distributions are covered (indexed) on Bintray.
But there are good news: since the package indexed for these distributions is identical,
any reasonably recent distribution name would suffice
in practice. For example, users of Debian Buster, Debian Sid, Ubuntu Disco and Ubuntu Eoan
can use both `stretch` and `bionic` for distribution name.

Below is a table of OS release and distribution names that should be used on Bintray.

| Release        | Distribution Name to Use on Bintray |
|----------------|-----------|
| Ubuntu 18.04   | `bionic`  |
| Ubuntu 19.04   | `bionic`  |
| Ubuntu 19.10   | `bionic`  |
| Ubuntu 16.04   | `xenial`  |
| Debian Buster  | `stretch` |
| Debian Stretch | `stretch` |
| Debian Sid     | `stretch` |


To add the apt repository to the source list directory (`/etc/apt/sources.list.d`), use:

```sh
echo "deb https://dl.bintray.com/rabbitmq/debian {distribution} main" | sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list
```

where `{distribution}` is the name of the Debian or Ubuntu distribution used (see the table above).

So, on Ubuntu 18.04 and later releases the above command becomes

```sh
echo "deb https://dl.bintray.com/rabbitmq/debian bionic main" | sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list
```

`bionic` for distribution name would also work on Debian Buster and Debian Sid.

On Debian Stretch it would be

```sh
echo "deb https://dl.bintray.com/rabbitmq/debian stretch main" | sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list
```

`stretch` for distribution name would also work on Debian Buster.

On Ubuntu 16.04 it would be

```sh
echo "deb https://dl.bintray.com/rabbitmq/debian xenial main" | sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list
```

It is possible to list multiple repositories, for example, one that provides RabbitMQ and one that [provides Erlang/OTP packages](#erlang-repositories).
On Ubuntu 18.04 that can be done by modifying the command in the above example like so:

```sh
sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list &lt;&lt;EOF
deb https://dl.bintray.com/rabbitmq-erlang/debian bionic erlang
deb https://dl.bintray.com/rabbitmq/debian bionic main
EOF
```

and on Ubuntu 16.04 it would be

```sh
sudo tee /etc/apt/sources.list.d/bintray.rabbitmq.list &lt;&lt;EOF
deb https://dl.bintray.com/rabbitmq-erlang/debian xenial erlang
deb https://dl.bintray.com/rabbitmq/debian xenial main
EOF
```

#### Install RabbitMQ Package

After updating the list of `apt` sources it is necessary to run `apt-get update`:

```sh
sudo apt-get update -y
```

Then install the package with

```sh
sudo apt-get install -y rabbitmq-server
```

## Manual Installation with Dpkg

In some cases it may easier to download the package directly from GitHub and install it manually using `sudo dpkg -i`.
Below is a download link.

<table>
  <thead>
    <th>Description</th>
    <th>Download</th>
    <th>Signature</th>
  </thead>

  <tr>
    <td>
      .deb for Debian-based Linux (from <a href="https://github.com/rabbitmq/rabbitmq-server/releases">GitHub</a>)
    </td>
    <td>
      <a href="https://github.com/rabbitmq/rabbitmq-server/releases/download/&version-server-tag;/rabbitmq-server_&version-server;-&serverDebMinorVersion;_all.deb">rabbitmq-server_&version-server;-&serverDebMinorVersion;_all.deb</a>
    </td>
    <td>
      <a href="https://github.com/rabbitmq/rabbitmq-server/releases/download/&version-server-tag;/rabbitmq-server_&version-server;-&serverDebMinorVersion;_all.deb.asc">Signature</a>
    </td>
  </tr>
</table>

When installing manually with `dpkg`, it is necessary to install package dependencies first.
`dpkg`, unlike `apt`, does not resolve or manage dependencies.

Here's an example that does that, installs `wget`, downloads the RabbitMQ package and installs it:

```sh
# sync package metadata
sudo apt-get update
# install dependencies manually
sudo apt-get -y install socat logrotate init-system-helpers adduser

# download the package
sudo apt-get -y install wget
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/&version-server-tag;/rabbitmq-server_&version-server;-&serverDebMinorVersion;_all.deb

# install the package with dpkg
sudo dpkg -i rabbitmq-server_&version-server;-&serverDebMinorVersion;_all.deb

rm rabbitmq-server_&version-server;-&serverDebMinorVersion;_all.deb
```

Installation via [apt repositories](#apt) on Bintray and Package Cloud is recommended
over downloading the package directly and installing via `dpkg -i`. When the RabbitMQ
package is installed manually with `dpkg -i` the operator is responsible for making sure
that all [package dependencies](#package-dependencies) are met.


## User Privilege Requirements

RabbitMQ Debian package will require `sudo` privileges to install and manage.
In environments where `sudo` isn't available, consider using the
[generic binary build](/install-generic-unix.html) instead.

## Run RabbitMQ Server

#### Start the Server

The server is started as a daemon by default when the
RabbitMQ server package is installed. It will run as a non-privileged user `rabbitmq`.

As an administrator, start and stop the
server as usual for Debian-based systems:
`service rabbitmq-server start`.


## Configuring RabbitMQ

On most systems, a node should be able to start and run with all defaults.
Please refer to the [Configuration guide](configure.html) to learn more
and [Production Checklist](/production-checklist.html) for guidelines beyond
development environments.

Note: the node is set up to run as system user `rabbitmq`.
If [location of the node database or the logs](/relocate.html) is changed,
the files and directories must be owned by this user.


## Port Access

RabbitMQ nodes bind to ports (open server TCP sockets) in order to accept client and CLI tool connections.
Other processes and tools such as SELinux may prevent RabbitMQ from binding to a port. When that happens,
the node will fail to start.

CLI tools, client libraries and RabbitMQ nodes also open connections (client TCP sockets).
Firewalls can prevent nodes and CLI tools from communicating with each other.
Make sure the following ports are accessible:

 * 4369: [epmd](http://erlang.org/doc/man/epmd.html), a peer discovery service used by RabbitMQ nodes and CLI tools
 * 5672, 5671: used by AMQP 0-9-1 and 1.0 clients without and with TLS
 * 25672: used for inter-node and CLI tools communication (Erlang distribution server port)
   and is allocated from a dynamic range (limited to a single port by default,
   computed as AMQP port + 20000). Unless external connections on these ports are really necessary (e.g.
   the cluster uses [federation](/federation.html) or CLI tools are used on machines outside the subnet),
   these ports should not be publicly exposed. See [networking guide](/networking.html) for details.
 * 35672-35682: used by CLI tools (Erlang distribution client ports) for communication with nodes
   and is allocated from a dynamic range (computed as server distribution port + 10000 through
   server distribution port + 10010). See [networking guide](/networking.html) for details.
 * 15672: [HTTP API](/management.html) clients, [management UI](/management.html) and [rabbitmqadmin](/management-cli.html)
   (only if the [management plugin](/management.html) is enabled)
 * 61613, 61614: [STOMP clients](https://stomp.github.io/stomp-specification-1.2.html) without and with TLS (only if the [STOMP plugin](/stomp.html) is enabled)
 * 1883, 8883: ([MQTT clients](http://mqtt.org/) without and with TLS, if the [MQTT plugin](/mqtt.html) is enabled
 * 15674: STOMP-over-WebSockets clients (only if the [Web STOMP plugin](/web-stomp.html) is enabled)
 * 15675: MQTT-over-WebSockets clients (only if the [Web MQTT plugin](/web-mqtt.html) is enabled)
 * 15692: Prometheus metrics (only if the [Prometheus plugin](/prometheus.html) is enabled)

It is possible to [configure RabbitMQ](/configure.html)
to use [different ports and specific network interfaces](/networking.html).


## Default User Access

The broker creates a user `guest` with password
`guest`. Unconfigured clients will in general use these
credentials. <strong>By default, these credentials can only be
used when connecting to the broker as localhost</strong> so you
will need to take action before connecting from any other
machine.

See the documentation on [access control](access-control.html) for information on how to create more users and delete
the `guest` user.


## Controlling System Limits on Linux

RabbitMQ installations running production workloads may need system
limits and kernel parameters tuning in order to handle a decent number of
concurrent connections and queues. The main setting that needs adjustment
is the max number of open files, also known as `ulimit -n`.
The default value on many operating systems is too low for a messaging
broker (`1024` on several Linux distributions). We recommend allowing
for at least 65536 file descriptors for user `rabbitmq` in
production environments. 4096 should be sufficient for many development
workloads.

There are two limits in play: the maximum number of open files the OS kernel
allows (`fs.file-max`) and the per-user limit (`ulimit -n`).
The former must be higher than the latter.

### With systemd (Recent Linux Distributions)

On distributions that use systemd, the OS limits are controlled via
a configuration file at `/etc/systemd/system/rabbitmq-server.service.d/limits.conf`.
For example, to set the max open file handle limit (`nofile`) to `64000`:

```ini
[Service]
LimitNOFILE=64000
```

See [systemd documentation](https://www.freedesktop.org/software/systemd/man/systemd.exec.html) to learn about
the supported limits and other directives.

### With Docker

To configure kernel limits for Docker contains, use the `"default-ulimits"` key in [Docker daemon configuration file](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file).
The file has to be installed on Docker hosts at `/etc/docker/daemon.json`:

```json
{
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  }
}
```

### Without systemd (Older Linux Distributions)

The most straightforward way to adjust the per-user limit for
RabbitMQ on distributions that do not use systemd is to edit the `/etc/default/rabbitmq-server`
(provided by the RabbitMQ Debian package) or [rabbitmq-env.conf](http://www.rabbitmq.com/configure.html)
to invoke `ulimit` before the service is started.

```sh
ulimit -S -n 4096
```

This <em>soft</em> limit cannot go higher than the <em>hard</em> limit (which defaults to 4096 in many distributions).
[The hard limit can be increased](https://github.com/basho/basho_docs/blob/master/content/riak/kv/2.2.3/using/performance/open-files-limit.md) via
`/etc/security/limits.conf`. This also requires enabling the [pam_limits.so](http://askubuntu.com/a/34559) module
and re-login or reboot. Note that limits cannot be changed for running OS processes.

For more information about controlling `fs.file-max`
with `sysctl`, please refer to the excellent
[Riak guide on open file limit tuning](https://github.com/basho/basho_docs/blob/master/content/riak/kv/2.2.3/using/performance/open-files-limit.md#debian--ubuntu).

### Verifying the Limit

[RabbitMQ management UI](management.html) displays the number of file descriptors available
for it to use on the Overview tab.

```sh
rabbitmqctl status
```

includes the same value.

The following command

```sh
cat /proc/$RABBITMQ_BEAM_PROCESS_PID/limits
```

can be used to display effective limits of a running process. `$RABBITMQ_BEAM_PROCESS_PID`
is the OS PID of the Erlang VM running RabbitMQ, as returned by `rabbitmqctl status`.

### Configuration Management Tools

Configuration management tools (e.g. Chef, Puppet, BOSH) provide assistance
with system limit tuning. Our [developer tools](devtools.html#devops-tools) guide
lists relevant modules and projects.


## Managing the Service

To start and stop the server, use the `service` tool.
The service name is `rabbitmq-server`:

```sh
# stop the local node
sudo service rabbitmq-server stop

# start it back
sudo service rabbitmq-server start
```

`service rabbitmq-server status` will report service status
as observed by systemd (or similar service manager):

```sh
# check on service status as observed by service manager
sudo service rabbitmq-server status
```

It will produce output similar to this:

```ini
Redirecting to /bin/systemctl status rabbitmq-server.service
● rabbitmq-server.service - RabbitMQ broker
   Loaded: loaded (/usr/lib/systemd/system/rabbitmq-server.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/rabbitmq-server.service.d
           └─limits.conf
   Active: active (running) since Wed 2018-12-26 10:21:32 UTC; 25s ago
 Main PID: 957 (beam.smp)
   Status: "Initialized"
   CGroup: /system.slice/rabbitmq-server.service
           ├─ 957 /usr/lib/erlang/erts-10.2/bin/beam.smp -W w -A 64 -MBas ageffcbf -MHas ageffcbf -MBlmbcs 512 -MHlmbcs 512 -MMmcs 30 -P 1048576 -t 5000000 -stbt db -zdbbl 128000 -K true -- -root /usr/lib/erlang -progname erl -- -home /var/lib/rabbitmq -- ...
           ├─1411 /usr/lib/erlang/erts-10.2/bin/epmd -daemon
           ├─1605 erl_child_setup 400000
           ├─2860 inet_gethost 4
           └─2861 inet_gethost 4

Dec 26 10:21:30 localhost.localdomain rabbitmq-server[957]: ##  ##
Dec 26 10:21:30 localhost.localdomain rabbitmq-server[957]: ##  ##      RabbitMQ 3.7.23. Copyright (c) 2007-2019 Pivotal Software, Inc.
Dec 26 10:21:30 localhost.localdomain rabbitmq-server[957]: ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
Dec 26 10:21:30 localhost.localdomain rabbitmq-server[957]: ######  ##
Dec 26 10:21:30 localhost.localdomain rabbitmq-server[957]: ##########  Logs: /var/log/rabbitmq/rabbit@localhost.log
Dec 26 10:21:30 localhost.localdomain rabbitmq-server[957]: /var/log/rabbitmq/rabbit@localhost_upgrade.log
Dec 26 10:21:30 localhost.localdomain rabbitmq-server[957]: Starting broker...
Dec 26 10:21:32 localhost.localdomain rabbitmq-server[957]: systemd unit for activation check: "rabbitmq-server.service"
Dec 26 10:21:32 localhost.localdomain systemd[1]: Started RabbitMQ broker.
Dec 26 10:21:32 localhost.localdomain rabbitmq-server[957]: completed with 6 plugins.
```

`rabbitmqctl`, `rabbitmq-diagnostics`,
and other [CLI tools](/cli.html) will be available in `PATH` and can be invoked by a `sudo`-enabled user:

```sh
# checks if the local node is running and CLI tools can successfully authenticate with it
sudo rabbitmq-diagnostics ping

# prints enabled components (applications), TCP listeners, memory usage breakdown, alarms
# and so on
sudo rabbitmq-diagnostics status

# prints effective node configuration
sudo rabbitmq-diagnostics environment

# performs a more extensive health check of the local node
sudo rabbitmq-diagnostics node_health_check
```

All `rabbitmqctl` commands will report the node absence if no broker is running.

See the [CLI tools guide](/cli.html) to learn more.


## Log Files and Management

[Server logs](/logging.html) can be found under the [configurable](/relocate.html) directory, which usually
defaults to `/var/log/rabbitmq` when RabbitMQ is installed via a Linux package manager.

`RABBITMQ_LOG_BASE` can be used to override [log directory location](/relocate.html).

Assuming a `systemd`-based distribution, system service logs can be
inspected using

```sh
journalctl --system
```

which requires superuser privileges.
Its output can be filtered to narrow it down to RabbitMQ-specific entries:

```sh
sudo journalctl --system | grep rabbitmq
```

The output will look similar to this:

```ini
Dec 26 11:03:04 localhost rabbitmq-server[968]: ##  ##
Dec 26 11:03:04 localhost rabbitmq-server[968]: ##  ##      RabbitMQ 3.7.16. Copyright (c) 2007-2020 Pivotal Software, Inc.
Dec 26 11:03:04 localhost rabbitmq-server[968]: ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
Dec 26 11:03:04 localhost rabbitmq-server[968]: ######  ##
Dec 26 11:03:04 localhost rabbitmq-server[968]: ##########  Logs: /var/log/rabbitmq/rabbit@localhost.log
Dec 26 11:03:04 localhost rabbitmq-server[968]: /var/log/rabbitmq/rabbit@localhost_upgrade.log
Dec 26 11:03:04 localhost rabbitmq-server[968]: Starting broker...
Dec 26 11:03:05 localhost rabbitmq-server[968]: systemd unit for activation check: "rabbitmq-server.service"
Dec 26 11:03:06 localhost rabbitmq-server[968]: completed with 6 plugins.
```

### Log Rotation

The broker always appends to the [log files](/logging.html), so a complete log history is retained.

[logrotate](https://linux.die.net/man/8/logrotate) is the recommended way of log file rotation and compression.
By default, the package will set up `logrotate` to run weekly on files located in default
`/var/log/rabbitmq` directory. Rotation configuration can be found in
`/etc/logrotate.d/rabbitmq-server`.
