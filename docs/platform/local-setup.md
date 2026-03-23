# Local Setup

This page describes preparing your local environment to run the tools you need for GovCMS, and then setting up a new
project.

Since we cater for a variety of developers with different experience, and many are working with limited tools in
government, we can't provide perfect documentation for everyone. If you have issues please refer to
[Getting help](../index.md#getting-help).

## Prerequisites

In order to run the local development environment, bundled in GovCMS, you need certain skills and experience in the
following:

* Using terminal to execute commands, launch programs, adjust filesystem ownership and permissions
* Installing and managing software packages
* Installing and debugging installation packages, via Ruby gem package manager
* Experience managing websites via Drush, executed in the terminal
* Experience managing Drupal configuration.

## Software Requirements

Supported platforms are:

* MacOS - 10.14 (Mojave) and above
* Windows 10 Pro

  !!! note
  Windows 10 users on version 1903 or higher can use Docker with WSL2, which offers performance improvements and,
  since it uses a full Linux environment, the commands are the same as those for Mac.

Before spinning up your local development environment, make sure the following software is installed and runs the latest
versions.

### Docker Desktop or Rancher Desktop

#### Docker Desktop

Use the [latest stable version](https://docs.docker.com/install/). It is available for macOS and Windows.

#### Rancher Desktop

[Rancher Desktop](https://rancherdesktop.io/) is the open source version of Docker Desktop and works in exactly the same
way.

It is safe to install both Docker and Rancher Desktop. But only one should be used at a time. That is, completely exit
one before using the other.

**Note**: Before starting with Rancher Desktop please read
[Recommended read about Rancher Desktop](https://www.drupaleasy.com/blogs/ultimike/2024/01/test-driving-rancher-desktop-docker-provider-ddev-macos).

If you are switching from Docker Desktop to Rancher Desktop or vice versa, you need to run one of the following commands.

Switching from Docker Desktop to Rancher Desktop:

```docker context use rancher-desktop```

Switching from Rancher Desktop to Docker Desktop:

```docker context use docker-desktop```

To list all your installed docker contexts:

```docker context ls```

### Pygmy

The latest instructions are available on the
[pygmystack GitHub repo](https://github.com/pygmystack/pygmy?tab=readme-ov-file#installation).

#### Linux & MacOS

This can also be installed on WSL-based systems using [Homebrew](https://brew.sh/).

```bash
brew tap pygmystack/pygmy;
brew install pygmy;
```

#### Windows

```bash
git clone https://github.com/pygmystack/pygmy.git && cd pygmy;
make build;
cp ./builds/pygmy-darwin /usr/local/bin/pygmy;
chmod +x /usr/local/bin/pygmy;
```

### Ahoy

[Ahoy (version 2)](https://ahoy-cli.readthedocs.io/en/latest/#version-2): currently MacOS and Linux only.

## Setting up your access and keys

If you are setting this up on the GovCMS platform, you will need access to the GovCMS Git Repository. Articles on how to
set that access up can be found on the GovCMS Knowledge Base.


## Typical local development workflow

### Start pygmy:

```pygmy up```

### Build and start the local environment

macOS: ```ahoy build```

Windows: ```docker compose up -d```

### Log in to the local GovCMS website

Once the build completes successfully, you can log in to the local copy of your GovCMS website by requesting a one-time
login link:

macOS and Linux: ```ahoy login```

Windows: ```docker compose exec -T test drush uli```

### Start or stop the local environment

You may start the local environment without building; however, we recommend to rebuild your environment every time you
work on a different git branch or task.

macOS and Linux: ```ahoy stop; ahoy start```

Windows: ```docker compose stop -d; docker compose start -d```

### Destroy and rebuild the local environment

macOS and Linux: ```ahoy down; ahoy build```

Windows: ```docker compose down -d; docker compose up -d```

## Update your local GovCMS version

!!! note
Before updating your local GovCMS version, we recommend you back up your local database with the following
command:
```ahoy mysql-dump [filename]```

GovCMS is available publicly as a pre-built Docker container. All you have to do is to run ahoy pull to download the
latest containers, followed by ahoy build to build the distribution:

macOS and Linux: ```ahoy pull```

Windows: ```docker image ls --format \"{{.Repository}}:{{.Tag}}\" | grep govcms/ | grep -v none | xargs -n1 docker pull | cat```

## Available ahoy commands

Using the terminal, and while inside your GitLab repo on your local, run the command `ahoy` without any parameters, to
get a list of all available commands. You can study the .ahoy.yml file in your GitLab repo, to see the internal docker
commands for each ahoy command (which is basically what you need to run on Windows).

## Troubleshooting and additional information

### Unable to install Pygmy on macOS

An attempt to install pygmy via the gem install pygmy command on macOS may result in the following error:

```You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory```

To resolve, run the following commands (this will install pygmy into an alternative directory):

```bash
gem update --system
gem install -n /usr/local/bin/ pygmy
```

### ERROR: Service 'cli' mounts volumes from 'amazeeio-ssh-agent', which is not the name of a service or container

If you see this error when you attempt to run any ahoy command, then you have not installed pygmy. See the "Software
Requirements" section at the top of this document.

### ERROR: failed to resolve source metadata for ...

If you get the following error when running ahoy build/up:

```bash 
ERROR [test internal] load metadata for docker.io/library/[sitename]:latest
target php: failed to solve: [sitename]: failed to resolve source metadata for docker.io/library/[sitename]:latest: pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
```

This is because the containers are built in parallel to each other, and some containers finish in the wrong order. To
fix this you can either build the CLI container first by running the following command first:

```docker compose build cli```

and then run:

```ahoy up```
OR
```ahoy build```

or you can direct composer not to use the new BAKE system by either running:

```COMPOSE_BAKE=false ahoy up```
OR
```COMPOSE_BAKE=false ahoy build```

Or apply the fix to your entire shell instance by running:

```export COMPOSE_BAKE=false```