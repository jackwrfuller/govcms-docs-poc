# Using the Lagoon CLI tool with GovCMS

The [Lagoon CLI](https://uselagoon.github.io/lagoon-cli/commands/lagoon/) allows PaaS users to interact with the GovCMS platform from the command line, and provides a method to control environment variables, deployments, and environments themselves.

## Link CLI with GovCMS Lagoon

```bash
lagoon config add \
--lagoon govcms \
--hostname ssh-lagoon.govcms.amazee.io \
--graphql https://api-lagoon.govcms.amazee.io/graphql \
--port 30831 \
--ui https://dashboard.govcms.gov.au \
--kibana https://logs-db-ui-lagoon.govcms.amazee.io
lagoon config default -l govcms
```

## Validate connectivity, show projects

```bash
lagoon list projects
```

[Read the docs](https://uselagoon.github.io/lagoon-cli/commands/lagoon/) for more information.
