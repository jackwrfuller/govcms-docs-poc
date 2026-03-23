# Debugging

## Setting up Xdebug with GovCMS SaaS

**Install Chrome Xdebug plugin.**

<https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc?hl=en>

### Config in Lagoon

1. Enable DOCKERHOST in .env.

    ```
    # Enable static dockerhost DNS
    DOCKERHOST=host.docker.internal
    ```

2. Rebuild docker images

    ```bash
    ahoy build cli
    # or
    ahoy build
    ```

3. Enable Xdebug in docker image

    ```bash
    ahoy debug
    ```

### Configure Visual Studio Code

1. Install the PHP Debug extension by Felix Becker.
2. Follow the instructions to create a basic `launch.json` for PHP.
3. Add correct path mappings. For a typical GovCMS SaaS project, an example would be:

    ```json
    {
      "version": "0.2.0",
      "configurations": [
        {
          "name": "Listen for Xdebug",
          "type": "php",
          "request": "launch",
          "port": 9000,
          "hostname": "localhost",
          "pathMappings": {
            "/app/config": "${workspaceFolder}/config",
            "/app/web/sites/default/files": "${workspaceFolder}/files",
            "/app/web/themes/custom/custom": "${workspaceFolder}/themes/custom"
          }
        }
      ]
    }
    ```

4. In the Run tab of Visual Studio Code, click the green arrow next to "Listen for Xdebug".
5. Enable Chrome Xdebug plugin on the listening page.
6. Load a webpage or run a Drush command.

### Optional: Sync application folder

```bash
ahoy my sync-app
```

Mapping app folder in `launch.json` `pathMapping` entries:

```json
"/app" : "${workspaceFolder}/app"
```
