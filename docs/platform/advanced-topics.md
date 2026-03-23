# Advanced Platform Topics

This is a collection of advanced topics which might help anyone who is doing extending or debugging the platform. It is possible for PaaS users to override a lot of default functionality (although not ideal) and this should serve as a primer.

## Bird's eye view

The [platform overview diagram](overview.md#diagram) is a good starting point to understanding the GovCMS/Docker/Lagoon architecture.

## Docker

The [Dockerfiles](https://github.com/govCMS/govcms8-scaffold-paas/tree/develop/.docker) that live in the .docker directory of a project are the cornerstone of a project. They contain the build logic which is shared in all environments.

**cli** ([Dockerfile.cli](https://github.com/govCMS/govcms8-scaffold-paas/blob/develop/.docker/Dockerfile.cli)): This image is built first. It is essentially a production friendly Drupal/PHP container, but no web requests are handled by this container (see `php` container below). There are no test tools in it. It can serve to allow command line access to Drupal without using the PHP container (for example, you can run drush commands in the **cli** container).

**php** ([Dockerfile.php](https://github.com/govCMS/govcms8-scaffold-paas/blob/develop/.docker/Dockerfile.php)): This image is built from the **cli** container with no practical differences. It will execute PHP as requested by Nginx.

**test** ([Dockerfile.test](https://github.com/govCMS/govcms8-scaffold-paas/blob/develop/.docker/Dockerfile.test)): This image is built from the CLI container, but gets loaded up with test tools. It does everything that the **cli** container does (you can run `drush`), but it also allows you to check coding standards or run Behat tests.

**nginx** ([Dockerfile.nginx-drupal](https://github.com/govCMS/govcms8-scaffold-paas/blob/develop/.docker/Dockerfile.nginx-drupal)): This image has Nginx and it responds to web requests. It has all the static files in `/app/web`, but if it receives a request to execute PHP (`index.php`) it will pass this request to the **php** container.

There are also other containers like **solr** which may be in use on your project. Note that `mariadb` is not built from a `Dockerfile`: it either uses a nightly backup image or the [generic image](https://hub.docker.com/r/govcms8lagoon/mariadb-drupal).

## Dockerfile build order

PHP, Nginx and Test all have a build dependency on CLI image. This is mainly so that we can copy the fully built `/app` directory from the CLI container into every other container.

* Dockerfile.**cli**
    * Dockerfile.**php**
    * Dockerfile.**nginx-drupal**
    * Dockerfile.**test**
* Dockerfile.**solr**
* Dockerfile.**...** (as applicable)

## Example Dockerfile

This describes a `Dockerfile` build and the logic behind each step.

```dockerfile
# CLI_IMAGE refers to the already built CLI image. Subsequent containers
# can copy directories and files from it. The build concept is the same
# on Lagoon, but while Lagoon injects the argument as a --build-arg,
# locally we inject the arg via docker-compose.yml.
ARG CLI_IMAGE

# A version tag may be passed in from .env or .env.default locally,
# or via GraphQL "buildtime" variable on Lagoon. It allows pulling
# images from alternative tags like :beta or :7.3.2. The `=latest`
# here only applies if the build argument is *not* set.
ARG GOVCMS_IMAGE_VERSION=latest

# Tells Docker to prep the CLI container for copying in assets.
FROM ${CLI_IMAGE} as cli

# The base image from https://hub.docker.com/u/govcms8lagoon
FROM govcms8lagoon/php:${GOVCMS_IMAGE_VERSION}

# Since we are building from a generic upstream govcms container, we
# effectively can't trust the files in it.
RUN rm -rf /app

# Copying files from the CLI is the main reason for providing it as
# a build argument.
COPY --from=cli /app/web /app/web
```

## docker-compose

Locally when you run `docker-compose up -d` you are executing the manifest defined in the [docker-compose.yml](https://github.com/govCMS/govcms8-scaffold-paas/blob/develop/docker-compose.yml). So this is how all Docker images are built and containers started (we never use `docker build ...`).

## Same-same but different

There are key differences between environments which can have a big impact on whether your cunning plan can be executed.

### docker-compose differences

Lagoon does **not** use `docker-compose` to build images and start containers. It will however inspect the `docker-compose.yml` to identify the containers to build but most of the configuration there is otherwise ignored. Any environment variables added to Gitlab will not be available in Lagoon.

### CircleCI differences

Our public repositories use CircleCI for testing. While CircleCI allows the use of `docker-compose`, it doesn't support the directory mounting techniques that we take advantage of locally and in Gitlab. This can lead to a lot of confusion as something that works locally or on Gitlab will not work in CircleCI. A benefit of how CircleCI works is that it forces you to be more thorough.

The important thing to understand here is that CircleCI treats Docker volumes exactly as Lagoon does in production: volumes are not shared with the host and only shared between containers. This means that all data should be added to the images during the build or into container during the runtime.

If you think "wait, if I use the machine executor for CircleCI, then I can use mounts", then think twice. Aside from this method being slower, and potentially incurring a cost in the future, it also means you can't use the [GovCMS CI image](https://github.com/govCMS/govcms-ci) which dramatically reduces the benefit.

For this and other differences when it comes to handling docker heavy builds, treat CircleCI and Gitlab as very different tools.

### Environment variable handling

The use of environment variables differs in a few places. They are the source of the what initially appear to neat configuration techniques, but later turn out to be somewhat obscure workarounds.

Locally, `docker-compose` will use your `.env` values so that is great.

In Gitlab, you can prepare environment variables in `gitlab-ci.yml` `VARIABLES`, however the Gitlab runner does not make `.env` variables available. In Gitlab, any the fancy variables in `docker-compose.yml` will just use the default value.

You will see in `docker-compose.yml` there are variables like `x-lagoon-project` to prepare values, but these are not used by Lagoon. Lagoon does does not "evaluate" the `docker-compose.yml`, it just parses it, so any Bash-like handling of default variable values is ignored by Lagoon. Lagoon doesn't use `.env` variables either, this sort of functionality is left to `lagoon.yml`.

In `docker-compose.yml`, you can see [CLI_IMAGE: *lagoon-project](https://github.com/govCMS/govcms8-scaffold-paas/blob/2f95b94/docker-compose.yml#L55) which is used [in the Dockerfile](https://github.com/govCMS/govcms8-scaffold-paas/blob/4769c51/.docker/Dockerfile.cli#L5). In Lagoon, `CLI_IMAGE`, `PHP_IMAGE`, `NGINX_IMAGE`, etc are prepared dynamically at build time as the ID of the container that was built - they are prepared in a completely different way.

### Database layer

For various reasons RDS (MySQL as a service on AWS) is used for Production sites, and in other environments either a "nightly backup" image, or a generic `govcms8lagoon/mariadb-drupal` is used. For this reason any performance testing of your database image configuration does not apply to your production instance.
