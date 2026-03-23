# Tips and Tricks

This is a collection of handy commands and tricks from govCMS users.

## Ahoy

### Adding your own custom commands to Ahoy

To add custom Ahoy commands, add this to your `.ahoy.yml` file:

```yml
me:
  usage: my stuff
  imports:
    - 'path/to/mycommands.ahoy.yml'
```

And then create a new file called `mycommands.ahoy.yml` in the same place as `.ahoy.yml`, and fill it out like this:

```yml
---
ahoyapi: v2
usage: Project specific ahoy commands.

commands:
  mygreatcommand:
    usage: Keep being great!
    cmd: echo 'Greatness level over 9000!!!'
```

You can now run your custom command with `ahoy me mygreatcommand`.

This is useful on SaaS since you can't commit the `.ahoy.yml` then it's annoying to lose your changes.

This also minimises the number of changes a colleague needs to make to start using my custom commands.

## Databases

### Emptying a database

To empty the database defined in `docroot/sites/default/settings.php` or `settings.local.php`, but not destroy the database itself, run this from your project root directory:

```bash
docker-compose exec cli drush sql-drop
```

### Importing a database

To import an existing database dump into a `mariadb` database inside your project's containers:

1. Placed an uncompressed dump of your database within your project's `docroot/` folder, then
2. Run this from your `docroot/` folder:

    ```bash
    docker-compose exec -T cli drush sql-cli < mydatabase.sql
    ```

    If you exclude the `-T` flag, Docker may complain that `the input device is not a TTY`. `docker-compose exec` runs with TTY allocation by default, the `-T` flag disables it.

    If Drush complains the database doesn't exist, you may need to stop and rebuild your containers to make Docker aware of the new database file.

## GovCMS7 Scaffolding PaaS

### Running your own Drupal site inside `govcms7-scaffold-paas`

GovCMS7 Scaffold PaaS ships without a website inside it's `/docroot` directory when you first clone it, so spinning it up results in a `No input file specified` error in the browser.

GovCMS scaffold also isn't set up to run a site locally, as the database details noted in the stock `settings.php` it ships with point to environment variables that aren't configured for local sites. But we can change that.

This next bit assumes you already have the `govcms7-scaffold-paas` repo up and running in Docker, and haven't made any changes.

If you want to run a new/existing Drupal site locally in the GovCMS7 Scaffold PaaS project:

**Importing your code**

1. From inside your `govcms7-scaffold-paas` project directory, ensure your Docker containers are stopped:

    ```bash
    docker-compose stop
    ```

2. Paste in your website files inside the `/docroot` directory. Don't just drop in the entire website root directory into `/docroot`, or the site will be sitting 1 level too low and won't load.

    It should look something like this:

    ```
    + govcms7-scaffold-paas/
      L .docker/
      + docroot/
        L modules/
        L scripts/
        L sites/
        L install.php
        ...
      L tests/
      L .ahoy.yml
      L .docker-compose.yml
      ...
    ```

    2.1. If you are importing an existing site, you'll want to place an uncompressed copy of your database under `docroot/` for importing later.

3. Create a new file called under `/docroot/sites/default` called `settings.local.php` and paste in the following code, taken from the database settings code block in `settings.php`:

    ```php
    <?php

    // Lagoon Database connection.
    if (getenv('LAGOON')) {
      $databases['default']['default'] = array(
        'driver' => 'mysql',
        'database' => 'drupal',
        'username' => 'drupal',
        'password' => 'drupal',
        'host' => 'mariadb',
        'port' => 3306,
        'prefix' => '',
      );
    }
    ```

4. Rebuild your containers:

    ```bash
    docker-compose up -d --build
    ```

5. Visit the site URL (defined under `/.docker-compose.yml` in the variable `&lagoon-local-dev-url`).

    5.1. If you've dropped in a new Drupal filebase, you should see a Drupal installation page; Install away, you're all done!

    5.2. If you've added an existing site, you should see Drupal errors because we haven't imported our existing site's database yet. Follow the instructions under [Importing a database](#importing-a-database).

6. Clear all caches:

    ```bash
    docker-compose exec -T test drush cc all
    ```

    Refresh your browser, your site should now be up and running inside PaaS!

## Pygmy

### Windows

Windows users have trouble with Pygmy more than anything else. There is a [Go version](https://github.com/fubarhouse/pygmy-go) in the pipeline to hopefully solve these issues. Otherwise try these steps:

### HTTPS

If you're using the experimental trial version of Pygmy written in Go, the following will configure Pygmy with HTTPS. However, the `/stats` page won't be available until the [respective PR](https://github.com/amazeeio/docker-haproxy/pull/5) is merged.

The following code snippet will need to be in the `$HOME/.pygmy.yml` file, and the URL in the scaffold/project `docker-compose.yml` will need to be changed to https.

```yaml
services:
  amazeeio-haproxy:
    HostConfig:
      PortBindings:
        443/tcp:
          -
            HostPort: 443
```

## Permissions for Admins on SaaS (GovCMS 8+)

SaaS by default restricts permissions on many basic admin Drupal functions, such as administering modules, permissions and viewing Site reports.

To re-enable some basic permissions for Administrators, run this in your CLI:

```bash
ahoy drush role-add-perm 'site_administrator' 'administer permissions, access site reports, administer modules'
ahoy drush en -y dblog
```

## Unblocking User 1 (GovCMS 8+)

To properly unblock User 1 locally, run this in the CLI:

```bash
ahoy drush uublk $(ahoy drush uinf --uid=1 --field=name)
ahoy drush ev '$u=\Drupal\user\Entity\User::load(1); $u->set("field_password_expiration", "0");  $u->set("field_last_password_reset", date("Y-m-d\TH:i:s")); $u->save();'
ahoy drush uli
```

## Using email locally

GovCMS ships with [Mailhog](https://github.com/mailhog) pre-configured, so when your containers are running, visit `https://mailhog.docker.amazee.io/` to access any emails sent by your local sites.

This is useful for testing webforms; give the webform a bogus 'to' address in the email handler, and when the form is submitted the resulting email will be 'hogged' by Mailhog without ever actually being sent anywhere.

Occasionally on Macs, emails may fail. See the [Troubleshooting](../platform/troubleshooting.md#mailhog-not-receiving-emails-sendmail-cannot-open-mxoutlagoonsvc465) page.

## Refresh remote site databases with PROD

There are several ways to refresh the database of a remote environment to match the latest PROD backup:

Delete the remote environments and:

1. **Easiest:** Re-trigger the `build` step in the deployment pipeline for that branch
2. Push an empty commit to re-trigger the deployment
3. Delete the remote _branch_ and re-push your local copy to re-trigger the deployment.
