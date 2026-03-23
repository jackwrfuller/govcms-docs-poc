# GovCMS SaaS Developer Guide

!!! note
    Please update the `govCMS/GovCMS` wiki on Github if you have any corrections or improvements.

This guide allows you to spin up a new govCMS Drupal 7 SaaS site locally using Docker, then import an existing SaaS site over it for local development. It covers using Windows, Linux or Mac OS, and assumes basic web developer knowledge and experience using a command line.

While anyone can create a new govCMS site via the official distro, importing a govCMS SaaS client site requires appropriate access to the site project repository to have already been granted by the Department of Finance.

**This involves having an account created by the govCMS team, with access to the appropriate projects.** Your project manager can request this by [lodging a support ticket with the GovCMS Support Desk](https://www.govcms.support) (preferred), or emailing support@govcms.gov.au.

**Running commands from this documentation**

When running commands listed in these instructions, anything wrapped in `${curly brackets like this}` represents a placeholder that you need to swap with your own information, so something like this:

```bash
cd ${my-project-name}
```

would actually be run as:

```bash
cd mygreatproject
```

To help ensure this guide works for people of all operating systems, some commands are shown twice, with the Ahoy version appearing first:

```
ahoy version:    ahoy nifty shortcut mycommand
full version:    the entire command in its elongated and transparent glory
```

Ahoy can normally be used for MacOS and Linux, but the full versions should work for everyone.

!!! warning
    Be sure to read the '[Known issues and workarounds](#known-issues-and-workarounds)' section at the end of this document. You might also want to check out the [Troubleshooting page](../platform/troubleshooting.md), it contains common errors and fixes.

## Official source material

You don't need to read it for this project, but sometimes it's nice to know the background :)

1. The vanilla GovCMS (7.x-3.x) Distribution is available at [Github Source](https://github.com/govcms/govcms) and as [Public DockerHub images](https://hub.docker.com/r/govcms)
2. Those GovCMS images are then customised for Lagoon and GovCMS, and are available at [Github Source](https://github.com/govcms/govcmslagoon) and as [Public DockerHub images](https://hub.docker.com/r/govcmslagoon)
3. Those GovCMS lagoon images are then retrieved by the `govcms-site` scaffold repository.

Documentation on debugging Docker for Windows issues lives on the [Amazeeio GitHub profile](https://github.com/amazeeio/docs/blob/master/local_docker_development/troubleshooting.md).

## Requirements

Version numbers are noted where applicable.

**Everyone**

* [Docker Desktop](https://docs.docker.com/install/) v18.09 min - A container management system. You only need the Community Edition, but will need to create a Docker account and note your Docker ID prior to using it.
* [Git](https://git-scm.com/downloads) v1.8 min - a code version control program. This can be installed either on its own (Mac/linux) or with [Git for Windows](https://gitforwindows.org/), or alternatively with a GUI such as [SourceTree](https://www.sourcetreeapp.com/). Mac users with Xcode or Xcode Command Line Tools installed may already have Git. Git also comes with [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

**Linux/Mac users**

* [Ruby](https://www.ruby-lang.org/en/) - A dynamic programming language, allows running of `pygmy`
* [pygmy](https://docs.amazee.io/local_docker_development/pygmy.html#installation) v0.9.10 min - A miniature local web server (You might need `sudo` for this depending on your ruby configuration)

**Windows users**

* Windows 10 Pro or Enterprise (Docker can't run on Windows Home edition as it doesn't come with Hyper-V)
* [Amazee Docker for Windows](https://github.com/amazeeio/amazeeio-docker-windows) - A set of additional Docker containers related to networking that run in Windows, in the absence of `pygmy`. Cloning instructions are in the '[Clone the project repo](#2-clone-the-project-repo)' section.

**Optional**

* A package manager (this will vary depending on your OS). Linux ships with one out of the box.
* [Ahoy](http://ahoy-cli.readthedocs.io/en/latest/#installation) - A shortcut manager for saving your precious fingers
* [NodeJS](https://nodejs.org/en/) - Allows the use of `npm`
* [Portainer](https://portainer.io/install.html) - A nice web UI for managing Docker containers

## Development rules

* You should create your theme(s) in folders under `/themes`
* Tests specific to your site can be committed to the `/tests` folders
* The files folder is not (currently) committed to GitLab.
* Do not make changes to `docker-compose.yml`, `lagoon.yml`, `.gitlab-ci.yml`, `.ahoy.yml`, `README.md`, or the Dockerfiles under `/.docker` - Altering these files will result in the project being unable to deploy to GovCMS SaaS. These files are locked in GitLab, so attempting to push changes will fail anyway.
* Some files can be extended/overridden by including additional files (see the '[Adding development modules](#adding-development-only-modules)' section)
* You should never need to change anything _inside_ a Docker container. Directly altering the contents of a container compromises it, meaning when a new instance of the project is spun up, those alterations won't be present.
* **Check for image updates every day!** To ensure you are running the latest environment, be sure to [pull down the latest images from the container registry](#image-updates) every day.

## Setting up a site

### 1. Connect to projects.govcms.gov.au

**Before you can clone anything from projects.govcms.gov.au, you must have an account created by the GovCMS team. Your project manager can request this by emailing support@govcms.gov.au**.

!!! note
    You may experience connectivity issues depending on your internet connection. Government internet connections run traffic through firewalls and gateways/filters, and block certain ports. SSH access often requires `port 22` to be available, and HTTPS often uses `port 443`.

You can either connect to `projects.govcms.gov.au` via HTTPS using a Personal Access Tokens (PAT) or via SSH using SSH keys. Using SSH keys is generally easier, as once set up they automatically authenticate you without needing to enter a password every time.

There's two scenarios where HTTPS is used:

- If you use HTTPS for the repository URL when cloning, and
- When [retrieving a copy of the production website database container from the GitLab Container Registry](#importing-a-database).

Check out the GitLab guides on:

* [creating and using SSH keys](https://docs.gitlab.com/ee/ssh/#generating-a-new-ssh-key-pair)
* [creating PATs](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) and [using them in Private Repositories](https://projects.govcms.gov.au/help/user/project/container_registry#using-with-private-projects)

If using SSH, you can test and debug your connectivity with this:

```bash
ssh -Tv git@projects.govcms.gov.au
```

### 2. Clone the project repo

Once you're all connected, navigate to wherever you want to keep the project and clone it down:

=== "SSH"
    ```bash
    git clone git@projects.govcms.gov.au:${profile_name}/${project_name}.git ${project_name}
    ```

=== "HTTPS"
    ```bash
    git clone https://projects.govcms.gov.au/${profile_name}/${project_name}.git ${project_name}
    ```

**Mac users only:** Confirm the project root path is in [Docker's file sharing config](https://docs.docker.com/docker-for-mac/#file-sharing).

**Windows users only:** In addition to the project repo, also clone the `amazee-docker-windows` repo:

```bash
git clone https://github.com/amazeeio/amazeeio-docker-windows amazee-docker-windows
```

### 3. Build it and run it

!!! warning
    Make sure you don't have anything running on port `80` on the host machine (like Apache, XAMP, WAMP, LAMP, or Skype).

Check what is running on port 80:

=== "Mac/Linux"
    ```bash
    sudo lsof -i -P -n | grep LISTEN
    ```

=== "Windows"
    ```bash
    netstat -anb | findstr :80
    ```

1. Ensure Docker is running. You can do this by running `docker container ps`.

2. Navigate into the project root.

3. Start the local network:

    === "Mac/Linux"
        ```bash
        pygmy up
        ```
        MacOS and Ubuntu 18 users may need to append `--no-resolver` if the site fails to load.

    === "Windows"
        Run from within the `amazee-docker-windows` root:

        ```bash
        # Create network (only needed once after Docker install)
        docker network create amazeeio-network
        # Start containers
        docker-compose up -d
        ```

4. Build and start the Docker containers (must be run from within the project root):

    ```bash
    ahoy up
    docker-compose up -d; docker-compose exec -T cli govcms-deploy
    ```

5. Install the GovCMS Drupal profile:

    ```bash
    ahoy install
    docker-compose exec -T test drush si -y govcms
    ```

    !!! note
        This must be done _before_ importing a backup of another site.

6. Update the user account password:

    ```bash
    ahoy drush user:password admin '${your-new-password}'
    docker-compose exec -T test drush user:password admin '${your-new-password}'
    ```

7. Login to your newly-installed Drupal site:

    ```bash
    ahoy login
    docker-compose exec -T test drush uli
    ```

## Image updates

Once your site is running correctly, you should check for updates to the images regularly.

=== "GovCMS 9/10 (SaaS)"
    ```bash
    ahoy pull
    docker image ls --format "{{.Repository}}:{{.Tag}}" | grep govcms/ | grep -v none | xargs -n1 docker pull | cat
    ```

=== "GovCMS 7"
    ```bash
    ahoy pull
    docker image ls --format "{{.Repository}}:{{.Tag}}" | grep govcmslagoon/ | grep -v none | xargs -n1 docker pull | cat
    ```

If still getting old images, you can also try pulling explicitly:

```bash
docker pull govcms/php:10.x-latest &&
docker pull govcms/nginx-drupal:10.x-latest &&
docker pull govcms/test:10.x-latest &&
docker pull govcms/govcms:10.x-latest &&
docker pull govcms/mariadb-drupal:10.x-latest &&
docker pull govcms/solr:10.x-latest &&
docker pull govcms/av:latest
```

??? info "Detailed instructions (including Windows)"

    All: (stops and deletes the running containers)

        docker-compose down

    Linux/Mac: (update stored images)

        ahoy pull

    Windows (cmd):

        docker image ls --format "{{.Repository}}:{{.Tag}}" | findstr "govcms/" | for /f %f in ('findstr /V "none"') do docker pull %f

    Windows (git bash):

        alias docker="winpty -Xallow-non-tty -Xplain docker"
        docker image ls --format \"{{.Repository}}:{{.Tag}}\" | grep govcms/ | grep -v none | awk "{print $1}" | xargs -n1 docker pull | cat
        alias docker="winpty docker"

    Windows (powershell):

        docker image ls --format "{{.Repository}}:{{.Tag}}" | Select-String -Pattern "govcms/" | Select-String -Pattern "none" -NotMatch | ForEach-Object -Process {docker pull $_}

    All: (build the new govcms containers from new images)

        docker compose build --no-cache

If any updates are found, you'll need to rebuild your containers with `docker-compose up -d --build`.

## Importing an existing site

You can install a base govCMS site from this project, then import the files and database of your production site.

!!! important
    - For imported backups to work, your local site must already have a vanilla govCMS site installed, otherwise you'll get a 503 error.
    - GovCMS Dashboard databases are automatically sanitized, so your normal login won't work! See the [Notes section](#notes) at the bottom.

### Importing a database

You can either import a local database file via Drush, or let Docker retrieve the latest production site database backup.

#### Importing a local database dump

1. Download a `mysql` container backup from the [govCMS Dashboard](https://dashboard.govcms.gov.au).

2. Extract the MySQL container backup:

    ```bash
    tar -xf ${backup_name}.tar
    ```

3. Navigate to the project location, and import the database:

    ```bash
    ahoy mysql-import ${path/to/my/local-database}.sql
    docker-compose exec test bash -c 'drush sql-drop' && docker-compose exec -T test bash -c 'drush sql-cli' < ${path/to/my/local-database}.sql
    ```

    !!! tip
        Importing via ahoy can be painfully slow. It runs much faster from inside the cli container:

        ```bash
        ahoy cli
        drush sql-cli < mydatabase.sql
        exit
        ```

    ??? info "Using PV to view database import progress"

        Install pv utility in docker:

            docker compose exec -T -u 0 mariadb apk add pv

        Import Gzipped SQL file (Linux/Mac/Windows WSL):

            sql_file=/path/to/database.sql.gz
            size=$(wc -c $sql_file | awk '{print $1}')
            cat $sql_file | docker compose exec -T mariadb sh -c "pv -f -s $size | gunzip | \
              awk 'NR == 1 && /\/\*M\!999999\\\- enable the sandbox mode \*\// {next} {print}' | \
              mariadb -u drupal -pdrupal drupal"

4. Flush the caches:

    ```bash
    ahoy drush cc all
    docker-compose exec -T drush cc all
    ```

#### Importing from the GitLab Container Registry

GitLab automatically saves the nightly production site database backups as containers in a private Container Registry.

1. Log into the Docker Container Registry with your GitLab Personal Access Token:

    ```bash
    docker login gitlab-registry-production.govcms.amazee.io -u ${your GitLab login email} -p ${your Personal Access Token}
    ```

2. In the file `.env`, uncomment the last line:

    ```bash
    MARIADB_DATA_IMAGE=gitlab-registry-production.govcms.amazee.io/${profile_name}/${project_name}/mariadb-drupal-data
    ```

    OR manually download:

    ```bash
    docker pull gitlab-registry-production.govcms.amazee.io/${profile_name}/${project_name}/mariadb-drupal-data
    ```

3. Rebuild the containers:

    ```bash
    ahoy up
    docker-compose up -d --build
    ```

### Exporting a database

```bash
ahoy mysql-dump ${path/to/my/local-database}.sql
docker-compose exec cli bash -c 'drush sql-dump --ordered-dump' > ${path/to/my/local-database}.sql
```

### Importing files

Files can be included in several ways:

1. Use the `stage_file_proxy` module to dynamically download files (preferred method); or
2. Download a dump of the files into the local file system.

#### To set up Stage File Proxy

The Stage File Proxy module is already included in govCMS SaaS, it just needs to be enabled:

=== "Drush"
    ```bash
    ahoy drush en -y stage_file_proxy
    docker-compose exec -T drush en -y stage_file_proxy
    ```

=== "govCMS script"
    ```bash
    docker-compose exec -T cli govcms-deploy
    ```

If you have already imported a SaaS production site database, Stage File Proxy should already have the internal production domain set as the source. No further action required!

If not, set it up manually:

=== "Drupal 7"
    ```bash
    ahoy drush variable-set stage_file_proxy_origin '${https://myproductionsite.com}'
    ```

=== "Drupal 8+"
    ```bash
    ahoy drush config-set stage_file_proxy.settings origin '${https://myproductionsite.com}'
    ```

## Adding development-only modules

For _new_ modules to work (ones not included in the govCMS installation profile), they need to be present in several Docker containers. Downloading them via Drush alone won't work.

To propagate a module across containers, set up additional Docker Volumes via a `docker-compose.override.yml` file.

!!! info
    This file is ignored by the GitLab deployment pipeline, so it cannot break anything if pushed to the remote repository.

1. Create a new file in the project root called `docker-compose.override.yml`:

    ```yml
    version: '2.3'

    services:
      cli:
        volumes:
          - ./dev_modules:/app/sites/default/modules/dev_modules:${VOLUME_FLAGS:-delegated}
      php:
        volumes:
          - ./dev_modules:/app/sites/default/modules/dev_modules:${VOLUME_FLAGS:-delegated}
      test:
        volumes:
          - ./dev_modules:/app/sites/default/modules/dev_modules:${VOLUME_FLAGS:-delegated}
    ```

2. Create a folder called `dev_modules` in the project root. **Remember to add it to gitignore.**

3. Rebuild the Docker containers:

    ```bash
    ahoy down; ahoy build
    docker-compose down; docker-compose up -d --build
    ```

4. Download the desired module:

    ```bash
    ahoy drush dl ${module_name} --destination='/app/sites/default/modules/dev_modules'
    docker-compose exec -T test drush dl ${module_name} --destination='/app/sites/default/modules/dev_modules'
    ```

5. Enable the module:

    ```bash
    ahoy drush en ${module_name}
    docker-compose exec -T test drush en ${module_name}
    ```

## Shutting down without losing work

Docker containers will remain intact unless you explicitly delete them.

!!! danger
    Running `ahoy down` or `docker-compose down` will **DESTROY** your containers and any changes within (importantly, the database). Use with caution!

To stop your containers for later use:

```bash
ahoy stop
docker-compose stop
```

To start them again later:

```bash
ahoy restart
docker-compose start
```

When the containers are stopped, you can safely shut down your computer without losing work. Changes made to files _outside_ the containers, such as in `/files` and `/themes` will remain intact if the containers are stopped or deleted.

## Pushing commits

Before pushing anything back up to GitLab, confirm your local Git user name and email:

```bash
git config --global --list
git config --local --list
```

You can specify different user details for specific repositories:

```bash
git config --local user.name '${your GitLab account username}'
git config --local user.email ${your GitLab account email}
```

## Dumping Variables

In Twig you can use `dsm()`

In PHP you can use `ksm()` for variable dump in status message or `kint()` for variable dump at place of execution.

### Advanced Dumping

When dumping deep structures, you may encounter Symfony's VarCloner limit. To increase minimum limit:

```php
$input = function_to_debug();
$cloner = new \Symfony\Component\VarDumper\Cloner\VarCloner();
$cloner->setMinDepth(10);
$dumper = new \Symfony\Component\VarDumper\Dumper\HtmlDumper();
$output = fopen('php://memory', 'r+b');
$data = $cloner->cloneVar($input);
$dumper->dump($data, $output);
$output = stream_get_contents($output, -1, 0);
$output = \Drupal\devel\Render\FilteredMarkup::create($output);
\Drupal::messenger()->addMessage($output);
```

## Running multiple copies of the same project

Currently the `container_name` in the `cli` container is not dynamic so you will get a naming conflict, even though you have set `COMPOSE_PROJECT_NAME` to be unique.

[See here](https://github.com/govCMS/scaffold/issues/126) for a proposed fix.

In the interim you can use the following by creating a `docker-compose.override.yml` file (rebuild of containers is required). Change `MYPROJECT` to your project name:

```yml
x-lagoon-project: &lagoon-project
  ${COMPOSE_PROJECT_NAME:-MYPROJECT}

services:
  cli:
    image: *lagoon-project
    container_name: !reset
  test:
    build:
      args:
        CLI_IMAGE: *lagoon-project
  nginx:
    build:
      args:
        CLI_IMAGE: *lagoon-project
  php:
    build:
      args:
        CLI_IMAGE: *lagoon-project
```

## Known issues and workarounds

1. This process only applies to the `7.x-3.x` branch of GovCMS
2. The container 'test' cannot have its name changed. This prevents Drupal from being able to connect to the database for some reason.
3. When logging into the site for the first time, the 'Reset password' page does not allow resetting the password. See [Step 6 of Setup](#3-build-it-and-run-it) for the workaround.

**Issues running Docker on Windows**

1. Windows users may get build errors when running `docker-compose up -d`, noting the filepaths listed in the dockerfiles are not valid. [The fix](https://community.quantrocket.com/t/errors-on-deploying-quantrocket-mount-denied-nthe-source-path-var-run-docker-sock-var-run-docker-sock-nis-not-a-valid-windows-path/258/3) is to add this line to the project's `.env` file: `COMPOSE_CONVERT_WINDOWS_PATHS=1`.

2. If Docker starts automatically with Windows, you may get an error like `Error starting userland proxy: mkdir /port/tcp:...`. This is a [known bug in Docker](https://github.com/docker/for-win/issues/1038) related to Windows Fast Boot. Either disable Fast Boot, or restart Docker once Windows loads.

3. If using Git Bash on Windows, you may get path misinterpretation errors. This is a [known bug in Docker](https://github.com/docker/toolbox/issues/673). Use another CLI like Command Prompt.

4. Even if you _disable_ the setting 'Start Docker Desktop when you log in', Docker may start at boot regardless. Check your Startup apps list (`Ctrl + Shift + Esc` > `Startup tab`).

## Notes

- Windows users can find the `docker` commands listed in this guide inside the `.ahoy.yml` file.
- Unless you import a database dump from another site, the out-of-the-box govCMS site will only contain the user 'admin'.
- GovCMS SaaS sites have the uid 1 account disabled, you can enable it on your local developer site by running:

    ```bash
    docker-compose exec -T cli drush user-information --pipe --uid=1 | xargs | awk '{print $2}' | xargs docker-compose exec -T cli drush user-unblock
    ```

- If you import a database dump from a site where your user account is NOT an administrator, you can become one by running:

    ```bash
    ahoy drush urol 'administrator' ${account email, user ID or 'username in quotes'}
    docker-compose exec -T test drush urol 'administrator' ${account email, user ID or 'username in quotes'}
    ```

- If you import a database from the govCMS Dashboard, it will be automatically sanitized. Emails and passwords will have changed, so to change them you must use the User ID or username.
- If a Docker build seems to be taking forever, check a password prompt from Docker hasn't opened somewhere requesting access to your hard drive.
