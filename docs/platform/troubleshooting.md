# Troubleshooting

This is a collection of issues encountered by govCMS users, and the ways they either got around them or died trying. If you've hit a problem you don't see here, add it in!

!!! tip
    Be sure to include any error messages, so users find this page when they google theirs!

## I can't log into my new local govCMS site, the password reset doesn't work!

GovCMS installs with a single-use login for the admin user, but refuses to allow setting a new password when prompted. This is a known bug.

Running `ahoy login` or `docker-compose exec -T test drush uli` may fail with the error `Password may only be changed in 24 hours from the last change`.

To fix it, reset the admin user password via Drush:

```bash
Mac/Linux:  ahoy drush upwd admin --password='${your-new-password}'
Windows:    docker-compose exec -T test drush upwd admin --password='${your-new-password}'
```

## `Provided sql file my-local-database.sql does not exist`

Sometimes when running `ahoy mysql-import <my-local-database>`, the ahoy command returns this error. No idea why.

The fix is to manually run the command inside the ahoy command:

```bash
ahoy drush sql-drop && ahoy drush sql-cli < my-local-database.sql
```

## `Command <my-command> needs a higher bootstrap level to run`

If you see this error when attempting to run a drush command:

```
Command <command> needs a higher bootstrap level to run - you        [error]
will need to invoke drush from a more functional Drupal environment
to run this command.
The drush command <command> could not be executed.              [error]
```

yet running `drush status` shows the website is alive, and the database connected successfully, I don't know what the problem really is.

The 'fix' is to blow away your local repo, download a fresh copy and `ahoy up`.

## Content images don't appear up on my local site

If you've imported a website, it's normal not to include the entire filebase as this won't be tracked in the website's Git repository.

Freshly imported sites will only include the database and tracked codebase, so images won't be there.

The [fix for this is detailed in the SaaS Developer Guide](../guides/saas-developer-guide.md#importing-files).

## Browser shows `No input file specified` error when visiting project's URL

This normally indicates that there is no website living inside your project, or if key files are missing from within it. So this could be resolved with `composer install`, which perhaps has failed to run properly on `ahoy up`.

In Drupal 7, when first running the `govcms7-scaffold-paas` project containers, and visiting `http://govms7-paas.docker.amazee.io`, this error will appear in the browser as the `/docroot/` folder doesn't contain a website, so the server doesn't know what to serve. To fix this, try:

1. Add a copy of Drupal (new site or existing site) into `/docroot`; OR
2. Run `docker-compose down`, then uncomment `line 31` of `.env` file and run `docker-compose up -d`
3. Check your local git repo for changes to files in the website folder (`/docroot/` for Drupal 7 or `/web/` for Drupal 8), such as missing/modified `index.php` or `.htaccess` files. Revert them and rebuild your containers with `docker-compose down` and `docker-compose up -d`

## My custom theme doesn't appear in the `Themes` admin page

If you've added a sub theme into your website's `theme` directory, but it isn't show up under `Admin > Appearance`, it may be a permissions issue preventing Docker from accessing the new files.

The fix is to:

1. Reset Docker Desktop to inherit and sharing permission changes:

    ```bash
    docker-compose stop
    docker-compose up -d
    ```

2. Then flush all your website caches:

    ```bash
    docker-compose exec -T test drush cc all
    ```

    or

    ```bash
    ahoy drush cc all
    ```

## No theme is rendering at all, just plain text on a white screen and errors about files!

This is normally caused by a missing `tmp` directory.

To fix it, create a new directory called `tmp` under `sites/default/files` (or wherever you have Drupal set to store it's temporary files).

## I get `input device is not a TTY` when running certain `docker-compose` commands

Issue on Mac OS and Windows.

`docker-compose exec` runs with TTY allocation by default. Sometimes, Docker complains that it doesn't want it when running certain commands, e.g:

```bash
docker-compose exec cli drush sql-cli < mydatabase.sql
the input device is not a TTY
```

To fix this, add a `-T` flag to the `exec` command:

```bash
docker-compose exec -T cli drush sql-cli < mydatabase.sql
```

**On Windows**, if the `-T` flag doesn't help, you may need to install [winpty](https://github.com/rprichard/winpty), which gives Unix-like powers to Windows console programs.

To use it, simply prefix your command with `winpty <command>`.

## Installing Pygmy fails

Discovered on MacOS.

When attempting to install `pygmy` with `gem install pygmy`, it may fail with this error (the path may vary based on your version of Ruby):

```
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions into the /usr/local/
```

[Installing the latest stable version of RVM (Ruby Version Manager)](https://rvm.io/rvm/install) can help avoid this. Be sure to use the command that installs the _stable_ version, not the _dev_ version!

## `pygmy up` failing

Discovered on MacOS.

Docker may be running on another user's profile. Try restarting your computer.

## Site cannot be found in browser after updating Docker

If you just updated Docker and your local site suddenly cannot be accessed via a browser, try turning `pygmy` off and on again - rebooting Docker messes with it.

```bash
pygmy down && pygmy up
```

## `docker login` failing with credentials error

Discovered on MacOS.

### Error in command line

```
Error saving credentials: error storing credentials - err: exit status 1, out: 'The user name or passphrase you entered is not correct.'
```

To fix this, open Keychain and lock/unlock your login keychain. Right click on the padlock.

## `docker pull` failing with 'Access Denied' error

### Error shown in command line

```
Using default tag: latest
Error response from daemon: Get https://gitlab-registry-production.govcms.amazee.io/v2/sandbox/_site_name_/mariadb-drupal-data/manifests/latest: unauthorized: HTTP Basic: Access denied
```

This is normally caused by not being logged into the GitLab registry where the images live. To fix it, log in with:

```bash
docker login gitlab-registry-production.govcms.amazee.io
```

Log in with your govCMS GitLab username (e.g. email address) and your Personal Access Token (PAT) as the password (setting up SSH access to GitLab will save you needing to do this repeatedly).

If you don't have a Personal Access Token (PAT), you may not have a GitLab account set up with GovCMS. GovCMS runs their own hosted instance of GitLab, which is separate from https://www.gitlab.com.

## `docker-compose up` failing when it reaches SOLR containers

Discovered on MacOS, while creating a govCMS8 site.

??? example "Issue - Instance 1"
    ```
    Creating network "gov8-site_default" with the default driver
    Creating gov8-site_mariadb_1 ...
    Creating gov8-site_solr_1    ... error
    Creating gov8-site_redis_1   ... done
    Creating gov8-site_mariadb_1 ... error

    ERROR: for gov8-site_solr_1  Cannot start service solr: driver failed programming external connectivity on endpoint... ...input/output error
    ```

??? example "Issue - Instance 2"
    ```
    Starting gov8-sitename_mariadb_1 ... error
    Starting gov8-sitename_solr_1    ... error

    ERROR: for mariadb  Cannot start service mariadb: driver failed programming external connectivity...
    ERROR: for solr  Cannot start service solr: driver failed programming external connectivity...
    ERROR: Encountered errors while bringing up the project.
    ```

**Solution 1:** Restart Docker.

**Solution 2:** Run `netstat -a -b` to see what was running on port `80` and kill the process via Task Manager; it was `httpd.exe`, i.e Apache.

## Drupal status report complains of an inconsistent database schema and mismatched tables

Discovered on MacOS after importing a PaaS site locally.

After signing in and viewing the Status Report, errors about database 4 byte UTF-8 support and inconsistent schema may appear.

Cause unknown, no fix yet available.

## Composer throws `Undefined index: extra` error when updating or requiring dependencies

```
[ErrorException]
  Undefined index: extra
```

This stems from [this issue](https://github.com/zaporylie/composer-drupal-optimizations/issues/18), and can be solved by deleting the file `./vendor/vendor/zaporylie/composer-drupal-optimizations`, then running `composer update zaporylie/composer-drupal-optimizations`

## Cannot connect to MySQL from my local CLI container

```
me@mymachine /app $ mysql -u drupal -pdrupal -h localhost drupal
Warning: World-writable config file '/etc/my.cnf.d/mariadb-client.cnf' is ignored
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/run/mysqld/mysqld.sock' (2)
```

This may stem from the credentials you use to access `mysql`. Anything referring to `localhost` or `127.0.0.1` inside a container will refer to that container itself, regardless of connected services.

Docker (for Mac) uses a special DNS name to talk to containers. Instead, use `docker.for.mac.host.internal`.

See <https://docs.docker.com/docker-for-mac/edge-release-notes/#docker-community-edition-17120-ce-mac45-2018-01-05>

Furthermore, you must also specify the `port` assigned to your `mariadb` container, which probably won't be `3306` like normal. To get a list of your container ports, run locally:

```bash
docker ps
```

```
ac932c650ad9   .../mariadb-drupal-data   "/sbin/tini -- /lago…"   19 hours ago   Up 19 hours   3306/tcp, 0.0.0.0:32797->3307/tcp
```

The port you need is the local one _mapped_ to `3307`, which is `32797`.

So to access MySQL, this should work instead when run inside your CLI container:

```bash
mysql -u drupal -pdrupal -h docker.for.mac.host.internal -P 32797 drupal
```

## InvalidArgumentException: Source path user has to start with a slash

FIX: This was caused by the `globalredirect` module - just remove it.

## Build fails with `toomanyrequests: You have reached your pull rate limit`

In November 2020, Docker started enforcing a pull limit on public images for anonymous users.

FIX: You need to be signed into Docker to avoid the pull limit. To do this on remote sites (i.e. lagoon), add this line to your project's gitlab-ci.yml file (check the spacing!):

```yaml
- docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_TOKEN
```

## In Connection.php line 416: SQLSTATE[HY000] [2002] Connection refused

This will occur if Drupal attempts to connect to a database with incorrect credentials - this can be a secondary database too, like when migrating sites.

If you are running a migration from Drupal 7 to 8 or 9, be sure to check any migration files that refer to database credentials. If a migration attempts to use the wrong details, you will see this error.

## Composer fails due to lack of memory

FIX: Set an alias to remove the PHP memory limit when calling Composer:

```bash
alias composer='php -d memory_limit=-1 /usr/local/bin/composer'
```

(use this path if Composer is installed globally. Otherwise, use whatever path points to your desired Composer installation).

By setting an alias, you don't need to reset this every time you open a terminal.

## Re-Importing migration config files shows a pile of changes, despite no changes

Normally, when importing config changes using `drush cim -y`, Drupal will run a diff to check for changes between the config in the files vs the config in the database.

When importing Migration config changes however, no diff is run, and all re-imported files will appear to contain updates even if they are identical to the config in the database.

## Content type cannot be edited, showing a WSOD error complaining of `language code 'und'`

The edit function may fail if govcms search (in particular search api) automatically enables the language module that allows content translation on the site. This has the detrimental result of breaking the edit interface because Drupal core locks 'und' langcode when language is enabled and throws an exception. You may also experience language-related errors during re-migration if migrating D7 - D8.

## Notice: tempnam(): file created in the system's temporary directory

This may appear if the private files directory is missing/misconfigured. Ensure your directory structure matches the filepath found under `/admin/config/media/file-system` and create the folders which might be missing. The default filepath for Drupal 8's private directory is `/sites/default/files/private/tmp`.

Permissions could be also be incorrect. Assuming the above directories exist, you can try `/bin/fix-permissions /app/web/sites/default/files`.

## WARN[0000] The GOVCMS_IMAGE_VERSION variable is not set. Defaulting to a blank string.

This error may appear when running `docker compose up -d --build`, after updating Docker along with this error stack:

```
WARN[0000] The GOVCMS_IMAGE_VERSION variable is not set. Defaulting to a blank string.
WARN[0000] The VOLUME_FLAGS variable is not set. Defaulting to a blank string.
WARN[0000] The DOCKERHOST variable is not set. Defaulting to a blank string.
WARN[0000] The XDEBUG_ENABLE variable is not set. Defaulting to a blank string.
services.php.environment.DEV_MODE must be a string, number or null
```

Commands that finally fixed it:

```bash
ahoy down
docker compose up -d --build
ahoy build
```

!!! note
    Related to docker compose v2 in 3.5.0 release:

    - docker compose v2 adds extra verbose messages (but continues to build as normal).
    - docker compose v2 now casts the DEV_MODE "true" to a boolean instead of string, causing fatal error. See <https://github.com/docker/compose-cli/issues/1903>

    Use `docker-compose disable-v2` to revert back to v1 functionality.

## Logging in locally after enabling TFA throws WSOD error

When logging into a local site with Two Factor Authentication (TFA) enabled and configured for your user profile (as is often the case with sites spun up from PROD backups), you may get this error after entering your password:

```
Drupal\encrypt\Exception\EncryptException: This encryption method requires a 32 byte key.
in Drupal\encrypt\EncryptService->validate() (line 115 of modules/contrib/encrypt/src/EncryptService.php).
```

To log in, you must either simply disable TFA as User 1 at `/admin/config/people/tfa`, or [configure TFA to run locally](https://www.govcms.support/support/solutions/articles/51000197889-how-to-use-test-tfa-locally). Note that the latter is not the most reliable method. It involves editing `docker-compose.yml`, which is a locked file in SaaS, and rebuilding your containers, and the changes should not be committed.

You can unblock User 1 and reset the password with Drush:

```bash
ahoy drush uublk $(ahoy drush uinf --uid=1 --field=name)
ahoy drush ev '$u=\Drupal\user\Entity\User::load(1); $u->set("field_password_expiration", "0"); $u->set("field_last_password_reset", date("Y-m-d\TH:i:s")); $u->save();'
echo "User 1 unblocked. Please reset the password with $ drush upwd <username> 'password'"
```

## D7 SaaS site loads with no theme, complaining of being unable to create files

If your D7 site loads with no CSS, and shows this error message:

`The file could not be created.`

If you see something like this inside the container when clearing the cache:

`file_put_contents(temporary://random-file-name): failed to open stream`

it may be that the permissions of `/sites/default/` is set to `555`. Try changing it with `chmod 755 /sites/default` and reload the page.

## Mailhog not receiving emails / `sendmail: Cannot open mxout.lagoon.svc:465`

Occasionally on MacOS, the port used to send mail within the Docker containers is blocked.

To confirm if mail can be sent, run this:

```bash
ahoy run 'echo "Subject: sendmail test" | sendmail -v my@email.com'
```

If the port is blocked, you will see: `sendmail: Cannot open mxout.lagoon.svc:465`

The current workaround is to explicitly assign the variable `SSMTP_MAILHUB` in either your `docker-compose.yml` or better yet a `docker-compose.override.yml` file:

```yaml
x-environment: &default-environment
  STAGE_FILE_PROXY_URL: ${STAGE_FILE_PROXY_URL:-}
  # Override: Add mailhub port to fix pygmy mapping issue
  SSMTP_MAILHUB: "host.docker.internal:1025"

  # On a MAC, you may need to use the IP address instead.
  # Run `ahoy cli` => `ping -c1 host.docker.internal` => Note IP address
  # then add instead:
  # SSMTP_MAILHUB: "XXX.XXX.XXX.XXX:1025"
```

Rebuild your containers, and test the mail again.
