# FAQ

We've recently updated some other docs and are deciding if a FAQ section would be useful. It might be a good place to add random pieces of information to keep the other pages simple. It also allows us to reference roadmap items. We welcome random items here, we might split them up later.

## GovCMS Distribution

**How do I get new modules into the GovCMS distribution?**

New additions to the distribution are challenging because there is a risk of bloating. Each additional module/version needs manual testing. On the roadmap is a process to allow vendors to request new modules and these be available from a whitelist of modules, rather than adding them to the distribution.

**I found a bug, is it Drupal or GovCMS?**

Normally in Drupal site building if you find a bug you can track this down to the Drupal.org issue queues. However there are two aspects of GovCMS which make it a little tricky to hunt down bugs.

GovCMS modules are usually a step or two behind the latest versions. This means that a bug that you find could already be fixed. Drupal developers (eg. peers in Slack) who are not using GovCMS may try to reproduce this bug but are unable, since they are working with an up-to-date codebase.

A practical bug-fixing workflow:

1. Assume the bug has already been fixed, search Drupal.org for the error text, ideally in the module issue queue.
2. Check that it's not a known issue or a known issue in the govcms [distro queue](https://github.com/govCMS/govCMS/issues)
3. Ask about the issue in GovCMS community channels (Slack, GovTEAMS)

## GovCMS Platform

**What certification does the platform have?**

The 2019 re-accreditation of the GovCMS platform has completed. Talk to the GovCMS team for further information.

**Can I use different docker images on the GovCMS platform?**

Yes, you can change your `docker-compose.yml` or your `Dockerfile.SERVICE` if you are a PaaS customer.

**Can we use a different hosting solution for GovCMS?**

You can host GovCMS distribution anywhere. Some advantages of hosting on the GovCMS platform are the helpdesk and the buying power around CDN services and DDOS protection.

**Should we use dblog (watchdog) in production?**

Best practice dictates the need to disable `dblog` module for performance reasons. So what if you don't have an alternative? You can enable dblog safely if you understand the implications.

If you turn on `dblog` manually in production, through the UI, it will be turned off on the next deploy - this will limit the long-term impact. Also, most of your pages are served from Akamai (CDN), and are not effected by whether `dblog` is on or off. When you enable `dblog`, you should be reviewing your log errors and fixing any issues that are apparent. If you need to leave the module on, please clear out log messages periodically.

## SaaS sites

[SaaS](https://github.com/govCMS/scaffold) is designed to be a controlled GovCMS/Drupal experience whereby the only customisations allowed are in your config and the theme. "SaaS" concepts straddle the vanilla distribution and the way we host it on Lagoon. Here are some frequently asked questions about SaaS on Drupal 8, 9 and 10.

[PaaS](https://github.com/govCMS/scaffold) sites are more like conventional Drupal sites, built with Composer. There is no "PaaS FAQ" because those topics are covered by Amazee/Lagoon or Drupal documentation.

**Can I have a custom admin theme?**

Yes, an admin theme is just a theme so it's allowed. It lives in your `/themes` folder next to your main theme.

**Can I customise my `settings.php`?**

On SaaS you can't customise settings.php (directly, or via an include) on the Lagoon platform. When you are working locally you can SSH into the PHP/CLI/TEST containers and modify `settings.php` file, or use some convenience command to automate this.

**Can I add custom modules on Lagoon?**

No, it's not possible on SaaS. From your repository, only `/themes` and `/config` are used in your Lagoon build.

**Can I add custom modules (eg `devel`) locally?**

Yes, you can add [docker-compose.override.yml](https://gist.github.com/simesy/765cc181503735c6784023073b9ba801) locally and mount some directories into your containers. You could put your modules in `custom/modules` and then mount this directory into `/app/web/modules/custom` or `/app/web/sites/default/local/modules`.

When you install modules that are not in the distribution, they will add new config to your site. You can see this when you run `ahoy drush en devel && ahoy drush cex && git diff`). This is not deployable so you need to be able to manage this: see [config_ignore](https://www.drupal.org/project/config_ignore) or [config_split](https://www.drupal.org/project/config_split).

**I notice my modules are not up to date, can I get the latest versions?**

No, we regression test whitelisted versions of all modules.

**Can I use a module patch with SaaS?**

No. We use some patches in the scaffold, but you can't alter these.

**Can I put PHP in my theme?**

Yes. You can implement hooks like `hook_form_alter()` and theme hooks like `hook_preprocess_node()`. You can also define name-spaced classes in `themes/mytheme/src` which can be auto-loaded.

You can't create new services (`mytheme.services.yml` and friends) as Drupal will not discover these, you can't implement things like routes, or override the services of other modules. There are a number of hooks which you could implement in Drupal 7, but they are module-only in Drupal 8, some of these hooks are [defined here](https://www.drupal.org/project/govcms8/issues/3047291#comment-13095172).

**Can I modify my roles and permissions?**

These are controlled in your config, so you can define roles and permissions locally and push these to the platform. Assigning an existing role to a user is different after forklift, so it's better to ensure there is a power user available prior to forklifting. There is a nightly audit on the platform which would remove or flag any permissions which are not appropriate.

**What is forklifting?**

It involves putting a copy of your database and `sites/default/files` up into the production environment. You should ensure that your database works with the master branch of your codebase, to avoid delays.

**Can I request a new module?**

Yes. However. Since this would need to be added to the distribution it's a difficult process. We regression test the distribution for each release and each new module makes this a harder process. We also have some issues with build performance given the size of the distribution. If you want to request a module, please be thorough and use this [issue template](https://github.com/govCMS/GovCMS/issues/new?template=module-request.md) to do so.

**Can I use the `minimal` or `standard` Drupal profile?**

Yes. Please be aware that any site building that is different to the GovCMS profile could make it harder for us to support departments who have no in-house technical support.

**I notice there is no `Publication` content type, can I add one?**

Yes. The GovCMS profile doesn't suit all needs and new content types, paragraph types and other entities will be added in many cases. Be aware that this requires site building expertise to control the visual display of your type content types - they may not magically work the same as the existing ones.

**Can I allow `<iframe>`s and `<script>`s in my text filters?**

Yes, however these are powerful tags which can cause issues if misused. We recommend creating an "Expert" text format, disabling the WYSIWYG for this format, and only allowing access to a role like Site Administrator.

**Can I customise my GitLab CI?**

No. However we have successfully tested a simple "CI switch" solution for PaaS which we hope to introduce to SaaS.
