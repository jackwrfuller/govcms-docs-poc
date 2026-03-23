# Variables

This page documents the environment variables and other variables used in the platform - specifically ones that GovCMS has created. You should check [Lagoon Variables](https://docs.lagoon.sh/concepts-advanced/environment-variables/) too.

If you see one of these variables in the wild, please ensure the documentation lives here, and update the variable comment in code to point to `https://govcms.gov.au/wiki_vars#VARIABLE_NAME`.

References to `.env` file also include `.env.default` file. (The `.env` file takes precedence). Lagoon also looks at `.lagoon.env` and `.lagoon.env.$BRANCH` files, but we are rarely (if ever) utilising them.

In the table below, a reference to just `.env` assumes it's used equally in all environments. References to "locally" are explicitly **not Lagoon**, however it probably includes GitLab jobs as these use Docker Compose. Bear in mind that GitLab variables can also be set through the GitLab UI, if you find a value is not working as expected.

| Variable | Locations | Description |
| --- | --- | --- |
| COMPOSE_PROJECT_NAME | .env locally | Site name and prefix for all containers. |
| DEV_MODE | .env locally | When set to true (DEV_MODE=true), Docker Compose uses this variable to enable Xdebug and Twig debug, and disables page caching and aggregation. |
| GOVCMS_IMAGE_VERSION | .env locally | Controls which version of GovCMS images are used, eg govcmslagoon/cli:latest vs govcmslagoon/cli:7.3.2. If unset this defaults to latest. This currently is only controlled in Lagoon through GraphQL as it controls a Docker buildtime variable. |
| LAGOON | Inside Lagoon containers | Lagoon [sets this](https://github.com/uselagoon/lagoon-images/blob/main/images/php-fpm/8.3.Dockerfile#L13) to match the name of the image. It's a reliable to determine if it's an image originating from Lagoon upstream. It is not used for checking "local vs remote". |
| LAGOON_ENVIRONMENT_TYPE | .env locally, Lagoon also sets | Lagoon sets this to "development" or "production". Locally it's forced to "local". Used by scripts to make decisions. |
| LOCALDEV_URL | .env locally | Local development URL |
| VOLUME_FLAGS | .env locally, OSX only | File access in hosted volumes can be very slow on Mac due to issues with the filesystem. Using `cached` or `delegated` here can really speed things up on OSX. |
| MARIADB_DATA_IMAGE | .env | An existing MariaDB-based Docker image. This will usually be your nightly database backup (created by the platform), but can be a generic pre-installed GovCMS. If this is not set, various processes like GitLab will install GovCMS Drupal profile by default _and not backup a nightly sanitized production database image to the gitlab registry_. |
| STAGE_FILE_PROXY_URL | .env | Used by Drupal for the [stage_file_proxy](https://www.drupal.org/project/stage_file_proxy) module. Point to alternate URL to ensure images and assets are available in the current environment. |
| X_FRAME_OPTIONS | .env | Controls a setting in Nginx container. Setting to `SameOrigin` disallows embedding content (e.g via iFrame) from any external domain. [Seckit](https://www.drupal.org/project/seckit) click-jacking configuration needs to be configured in parallel to this setting. |
| GOVCMS_DEPLOY_WORKFLOW_CONFIG | .env locally | Controls configuration management strategy on deploy. `retain` will retain existing configuration in the database and skip config import. `import` will import configuration from `config/default` (provided files are present in the codebase). **Note:** A support desk ticket is required to alter configuration strategy on platform. |
