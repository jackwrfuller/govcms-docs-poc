# File Structure

Generic file structure of the SaaS site:

```
.
├── .docker
│   ├── Dockerfile.cli
│   ├── Dockerfile.nginx-drupal
│   ├── Dockerfile.php
│   ├── Dockerfile.solr
│   └── Dockerfile.test
├── config
│   ├── default
│   ├── dev
│   │   └── README.md
│   └── sync
├── custom
│   └── ahoy.yml
├── drush
│   ├── sites
│   │   └── feature.site.yml
│   └── drush.yml
├── files
│   └── private
│       └── tmp
│           └── .gitkeep
├── scripts
│   └── composer
│       └── ScriptHandler.php
├── tests
│   ├── behat
│   │   ├── bootstrap
│   │   ├── features
│   │   │   ├── .gitkeep
│   │   │   └── home.feature
│   │   └── screenshots
│   │       └── .gitkeep
│   ├── phpunit
│   │   └── tests
│   │       ├── .gitkeep
│   │       └── ExampleTest.php
├── themes
│   └── .gitkeep
├── .ahoy.yml
├── .dockerignore
├── .editorconfig
├── .env
├── .env.default
├── .gitattributes
├── .gitignore
├── .gitlab-ci.yml
├── .lagoon.env
├── .lagoon.env.master
├── .lagoon.yml
├── .version.yml
├── behat.yml
├── docker-compose.yml
├── favicon.ico
├── README.md
└── redirects-map.conf
```

See the full list of files in the [govCMS/scaffold](https://github.com/govCMS/scaffold) project.

## Locked files (GovCMS SaaS)

Some of the files in the repository are "locked" - when any changes to these files pushed to the repository, these changes will be declined and the git push will fail. The "lock" on these files is set by the platform team because they are considered to be internal files required to make the platform function as expected.

Here is a list of "locked" files:

```
.docker/*
.ahoy.yml
.env.default
.gitlab-ci.yml
.lagoon.yml
.version.yml
README.md
docker-compose.yml
```
