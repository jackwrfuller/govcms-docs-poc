# Project Scaffold

This page explains the GovCMS [project scaffold](https://github.com/govCMS/scaffold) strategy for Drupal 8, 9 and 10. It describes how to modify a site at the scaffold level, while maintaining compatibility with upstream changes.

## It's just Drupal

As much as possible, the starting point for GovCMS is a standard Drupal site. The canonical pattern for setting up Drupal with composer is [drupal-composer/drupal-project](https://github.com/drupal-composer/drupal-project) or the new super trim [drupal/recommended-project](https://github.com/drupal/recommended-project).

If you want to create a new GovCMS project you can run the `composer create-project` command below. Just ignore the Lagoon/Docker configuration and the result is just Drupal with the GovCMS profile.

```bash
composer create-project https://github.com/govCMS/scaffold MYPROJECT
```

## Key composer packages

The `composer.json` is intentionally simple because the ideal state for the hosting platform is that all sites have the same `composer.json` file. PaaS developers can modify this file but have the option to keep it up to date with upstream changes.

```json
"require": {
    "govcms/govcms": "~1",
    "govcms/scaffold-tooling": "~1",
    "govcms/custom": "*"
},
"require-dev": {
    "govcms/require-dev": "~1"
},
```

| Package | Description |
| --- | --- |
| [govcms/govcms](https://github.com/govCMS/GovCMS) | The GovCMS Drupal distribution. |
| [govcms/scaffold-tooling](https://github.com/govCMS/scaffold-tooling) | Scripts, PHP settings, packages that are not in the distro. Installed to `vendor/govcms/scaffold-tooling` |
| [govcms/custom](https://github.com/govCMS/scaffold/tree/develop/custom/composer) | Points the `custom/composer` directory, as is a place to define custom composer packages and patches. Mostly applicable to PaaS sites. |

## Drupal Settings

The scaffold has a single [settings.php](https://github.com/govCMS/scaffold-tooling/blob/10.x-develop/drupal/settings/settings.php) file which includes additional files. We manage most of these files in [govcms/scaffold-tooling](https://github.com/govCMS/scaffold-tooling/tree/develop/drupal/settings) so that they are the same for all sites by default.

| Include | Where | What it does |
| --- | --- | --- |
| `all.settings.php` | [scaffold-tooling](https://github.com/govCMS/scaffold-tooling/blob/10.x-develop/drupal/settings/all.settings.php) | Universally ideal default settings for GovCMS clients |
| `lagoon.settings.php` | [scaffold-tooling](https://github.com/govCMS/scaffold-tooling/blob/10.x-develop/drupal/settings/lagoon.settings.php) | Ideal settings for running in Lagoon containers |
| `development.` OR `production.settings.php` | [scaffold-tooling](https://github.com/govCMS/scaffold-tooling/blob/10.x-develop/drupal/settings/development.settings.php) | Ideal settings for production VS non-production sites |
| `project.settings.php` | `web/sites/default` | Optional, lives in Git, specific for your project. |
| `local.settings.php` | `web/sites/default` | Local file not in Git, usually custom for a developer |

## Miscellaneous Scripts

After you have installed Drupal, there are a bunch of scripts in `vendor/bin` which we use for testing. Our scripts are prefixed with `govcms-*`. You don't need to use these scripts but they do cover a lot of standard testing scenarios and show how to do linting and other tasks with the tools in the distribution.

The scripts live in [govcms/scaffold-tooling](https://github.com/govCMS/scaffold-tooling/tree/10.x-develop/scripts) but beware that some rely on packages installed by [govcms/require-dev](https://github.com/govCMS/require-dev).

## Gitlab CI

The primary Gitlab CI file actually lives in [govcms/scaffold-tooling](https://github.com/govCMS/scaffold-tooling/blob/10.x-develop/.gitlab-ci-main.yml) so that we can manage it in one place.

You can enable (or disable) a range of jobs in Gitlab. Enabling every job would be redundant and would add a lot of time to your build. The types of jobs available are:

| Job | Setup | Description |
| --- | --- | --- |
| Lint | Fast, no Docker | Linting. On by default. |
| Unit | Fast, no Docker | PHPunit testing. Off by default. |
| Vet | Fast, no Docker | You can't disable this job. It fails SaaS builds if non-compliance detected. |
| Preflight | Drush level | Ensures the site can be built and `drush status` works. |
| Audit | Slow, full Docker | Off by default. Can run manually. On for merge. |
| Behat | Slow, full Docker | Full behavioural testing. Off by default. |
| Functional | Slow, full Docker | These are for PHPUnit driven [Drupal test traits](https://gitlab.com/weitzman/drupal-test-traits) which are an alternative to Behat testing. Off by default. |
| Deploy | Fast, no Docker | Simply offers a link to the Lagoon dashboard. |

There is a special switch file [.gitlab-ci-inputs.yml](https://github.com/govCMS/govcms8-scaffold-paas/blob/develop/.gitlab-ci-inputs.yml) that allows you to turn jobs on and off. The following settings will turn off a job without failing the build. These settings are under SaaS control.

```yaml
.job-EXAMPLE:
  when: manual
  allow_failure: true
```

You may want to enable a job, but not fail the build if the job fails. This is useful if you are slowly working through your linting errors, or if you have an unexpected failure but you are confident that it won't break your site.

```yaml
.job-EXAMPLE:
  when: on_success
  allow_failure: true
```

To fail a job on error set `allow_failure: false`.

```yaml
.job-EXAMPLE:
  when: on_success
  allow_failure: false
```

## How tos

```php
/**
 * @file
 * Drupal settings entry point.
 *
 * Do you need to override or set defaults? Check the stories below:
 *
 * As a developer ...
 *
 * `I want to override some settings just for me.
 *
 *   Create a web/sites/default/settings.local.php (this isn't committed to
 *   Git). There an example.settings.local.php to get you started.
 *
 * `I want to override some settings for this project everywhere it runs.
 *
 *   You can update web/sites/default/settings.project.php. See that file for
 *   more information.
 *
 * `I want to run an alternative local development environment.
 *
 *   You can use "project.settings.php" to add any variables you need. Or better
 *   to include an additional file eg web/sites/default/lando.settings.php. Make
 *   sure the LAGOON environment variable is not set. And set
 *   LAGOON_ENVIRONMENT_TYPE to 'local'.
 *
 * `I don't like these settings, my project is PaaS, and I want more control.
 *
 *   Have a look at lagoon.settings.php to understand what needs to be set to
 *   run a site on Lagoon. Our recommendation is that you include this file and
 *   override it at the project.settings.php level.
 *
 * `I feel like there are some settings that should be the same for every
 *  GovCMS site.
 *
 *   We welcome your observations!
 */
```
