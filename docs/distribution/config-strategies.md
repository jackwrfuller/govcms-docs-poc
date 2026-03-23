# Config Strategies

This page is about content and configuration in Drupal 8, 9 and 10.

Configuration for a Drupal site can be stored in `.yml` files (in the directory `config/default`). When a GovCMS site is deployed, the config from these `.yml` files is imported into the site. This overrides any configuration in the database.

## Config deployments

The default (as of 2020) for new GovCMS sites is that config is **not** deployed from code. This means that any configuration changes made to the production site will remain in place after a code deployment.

SaaS developers who wish to deploy configuration from code may request this change via the service desk. PaaS developers can make this change directly by [uncommenting this line](https://github.com/govCMS/govcms8-scaffold-paas/blob/develop/.lagoon.yml#L59).

Locally, there is the `govcms-deploy` script (which runs during commands like `ahoy up`). This script will try to import config from code if it can. To ensure that a deployment will never overwrite config in your local site, delete all the `.yml` files in `config/default`. If you **want** to take advantage of "config as code" follow the steps below.

## Create a config snapshot

If you are not deploying config you can still create a snapshot of your config at a given point in time. Run `ahoy drush cex --destination=../config/snapshot`, then your commit message can be something like "Creating a config backup that is not deployed automatically."

Later you can use this snapshot to see what config has changed in production.

## Advanced config breakdown

A running Drupal site consists of three parts: content, config and code. If you have these three components, you can reproduce any Drupal site in another environment.

Here's a breakdown of the three aspects:

* Content lives in the database.
* Code lives in the the git repository.
* Config lives in **both** the file system (`config/default`) and in the database, and can be synchronised between the two.

```
               A drupal site

           - +---------------------------------+
          |  |                 nodes     files |
          |  | *content*       users     terms |
          |  |                 blocks          |
   Lives  |  |                 paragraphs      |
      in -+  +---------------------------------+ -
database  |  |                 views           |  |
          |  | *config*        settings        |  |
          |  |                 webforms        |  |  Lives
           - +---------------------------------+  +- in
             |                 php             |  |  code
             | *code*          twig            |  |
             |                 css             |  |
             +---------------------------------+ -
```

In a running Drupal site, the config is loaded from the database. But to *deploy* config changes between environments, we use the file system (the files in the `config/default` directory).

GovCMS allows you to deploy code using git, with a `git push` or through a Gitlab merge request. This process also deploys your configuration. This happens whenever you push code to a branch.

!!! warning
    There is a risk of overwriting config when you deploy code. If you have editors working on the site you can revert some changes unexpectedly. See "Exporting config to code" section below if you want to ensure that you are capturing changes in a production site.

## Exporting config to code

Anytime you work with a GovCMS site, you should establish if the site is doing config deployments, and then make sure that any configuration in the Production site has been exported to `config/default`. Exporting this config to code will ensure that you don't reverse any Production settings when you deploy your other changes.

The general steps are:

1. Create a branch from `master`
2. Get a copy of the production database
    * from a nightly backup: in `.env` uncomment `MARIADB_DATA_IMAGE` then

        `ahoy refresh-db`    or

        `docker-compose pull mariadb`

    * from a database dump: in dashboard tasks select "Generate database backup [drush sql-dump]" save and extract dump to repository then

        `ahoy mysql-import ./database-name.sql`  or

        `docker-compose exec -T cli drush sql-drop` followed by

        `docker-compose exec -T cli drush sqlc < database-name.sql` (requires git-bash in Windows).

3. Export the config (either `ahoy drush config-export` or `docker-compose exec -T cli drush cex sync`).
4. Commit any changes in `/config/default`
5. Ensure the Config Ignore settings (`/admin/config/development/configuration/ignore`) aren't set to ignore all config (`*`), preventing anything being imported.
6. Push to Gitlab, merge request to `master`.

## Importing config on remote environments

If config management has been enabled on your site, but deployed config changes aren't appearing, check your config's `ignore` settings (`/admin/config/development/configuration/ignore`).

Sometimes this will include a wildcard at the top (`*`), which ignores all config when a deployment runs. SaaS sites having config management enabled for the first time sometimes have this accidentally left there.

You'll need to lodge a support ticket with GovCMS to have the wildcard removed on ALL environments - otherwise config will never deploy to them.
