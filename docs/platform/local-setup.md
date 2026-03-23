# Local Setup

This page describes preparing your local environment to run the tools you need for GovCMS, and then setting up a new project.

Since we cater for a variety of developers with different experience, and many are working with limited tools in government, we can't provide perfect documentation for everyone. If you have issues please refer to [Getting help](../index.md#getting-help).

## Dependencies

If your site is using the GovCMS platform, and you want to develop this site locally, you will need these tools.

* [Git](https://git-scm.com/downloads)
* Docker & docker-compose [Windows](https://docs.docker.com/docker-for-windows/install/), [OSX](https://docs.docker.com/docker-for-mac/install/)
* [Pygmy](https://pygmystack.github.io/pygmy/local_docker_development/)
* [Ahoy](http://ahoy-cli.readthedocs.io/en/latest/#installation) (recommended)

## Note for Windows

We are trying to improve the Windows experience. Most commands need to be carried out in Powershell but some may require [Git-bash](https://gitforwindows.org/). Use both tools with elevated permissions. DOS prompt is not appropriate for local development.

See also the [Lagoon docs](https://docs.lagoon.sh/using-lagoon-the-basics/local-development-environments/) as well as the Windows-specific tips in the [SaaS developer guide](../guides/saas-developer-guide.md).

## Upcoming opportunities

There is a [trial version of Pygmy](https://github.com/fubarhouse/pygmy-go) written in Go.

This will help to address many of the common issues users have had with Pygmy in the past including:

- Windows support (which may help Windows users who cannot install Ruby gems at present).
- Granular control over ports and services (run Pygmy on HTTPS with no effort; run phpmyadmin/portainer as well)

### Instructions to install Pygmy-go (Mac/Linux/WSL)

1. Clone the pygmy-go repo

    Change to directory. e.g. `cd /Users/[username]/programs`
    (replace [username] with your username)

    Run `git clone https://github.com/pygmystack/pygmy.git pygmy-go`

    Change to directory to pygmy-go. e.g.

    * `cd /Users/[username]/programs/pygmy-go` (Mac)
    * `cd /home/[username]/programs/pygmy-go` (Linux/WSL)

2. Compile pygmy-go to make pygmy executable

    !!! note
        * If you have a SSL scanning program, e.g. zscaler by your organization, ensure the extra ssl cert is added to docker file first to avoid error "unable to get local issuer certificate"!
        * If you get error `reading https://proxy.golang.org/golang.org/[projectfile]: 403 Forbidden`, then add `ENV GOPROXY="https://goproxy.io,direct"` prior to `WORKDIR` instruction in the `Dockerfile`. and try again a few times.

    For Zscaler WSL, see title "Instructions for Zscaler (windows)".

    ??? info "Instructions for Zscaler (mac)"

        **Option 1 (export PEM)**

        MAC - export certificate:

        1. Open the MAC Settings app
        2. GO to KeyChain Access
        3. GO to System / System roots
        4. Find Zscaler Root SSL
        5. Export the SSL file, Set the file format as "Privacy Enhanced Mail (.pem)" to [pygmy folder] -> e.g. 'zscaler.pem'.
        6. Replace these lines in the `[pygmy folder]/Dockerfile`:

        ```
        COPY external/ /go/src/github.com/pygmystack/pygmy/external/

        WORKDIR /go/src/github.com/pygmystack/pygmy/
        RUN GO111MODULE=on go mod verify
        ```

        With:

        ```
        COPY external/ /go/src/github.com/pygmystack/pygmy/external/

        COPY 'zscaler.pem' '/tmp/zscaler.pem'
        RUN CACHE_BUST_COUNTER=1 && cp '/tmp/zscaler.pem' '/usr/local/share/ca-certificates/zscaler.pem'
        RUN update-ca-certificates

        WORKDIR /go/src/github.com/pygmystack/pygmy/
        RUN GO111MODULE=on GOPROXY='proxy.golang.org,direct' go mod tidy
        RUN GO111MODULE=on go mod verify
        ```

        **Option 2 (export CER)**

        MAC - export certificate:

        1. Open the MAC Settings app
        2. GO to KeyChain Access
        3. GO to System / System roots
        4. Find Zscaler Root SSL
        5. Export the SSL file, Set the file format as "Certificate (.cer)" to [pygmy folder] -> e.g. 'zscaler.cer'.
        6. Replace these lines in the `[pygmy folder]/Dockerfile`:

        ```
        COPY external/ /go/src/github.com/pygmystack/pygmy/external/

        WORKDIR /go/src/github.com/pygmystack/pygmy/
        RUN GO111MODULE=on go mod verify
        ```

        With:

        ```
        COPY external/ /go/src/github.com/pygmystack/pygmy/external/

        RUN apk update
        RUN apk add curl openssl
        RUN CACHE_BUST_COUNTER=1 && echo "Cache bust counter: $CACHE_BUST_COUNTER"
        COPY zscaler.cer /tmp/
        RUN openssl x509 -inform DER -in '/tmp/zscaler.cer' -out '/usr/local/share/ca-certificates/zscaler.crt'
        RUN update-ca-certificates

        WORKDIR /go/src/github.com/pygmystack/pygmy/
        RUN GO111MODULE=on GOPROXY='proxy.golang.org,direct' go mod tidy
        RUN GO111MODULE=on go mod verify
        ```

        See <https://help.zscaler.com/zia/adding-custoem-certificate-application-specific-trusted-store#docker-file>

3. Run `make build`

    ??? tip "Debug if pygmy-go build fails"
        Edit `[pygmy folder]/Makefile`

        Replace `docker build -t pygmy .`

        With `docker build --no-cache -t pygmy --progress=plain .`

        and try again to see the error message.

4. Add pygmy to PATH and make executable.

    Change directory to the builds folder:
    e.g.

    * `cd /Users/[username]/programs/pygmy-go/builds` (Mac)
    * `cd /home/[username]/programs/pygmy-go/builds` (Linux/WSL)

5. Copy the correct executable file to the user binaries folder.

    e.g. `cp ./builds/[pygmy-file] /usr/local/bin/pygmy`

    Where [pygmy file] is:

    * `pygmy-darwin-arm64` (M1 (arm) Mac)
    * `pygmy-linux-amd64` (Linux / WSL - most Linux OSs are 64bit)
    * `pygmy-darwin-amd64` (Intel Mac)

    So for example:

    * for an M1 Mac, run `sudo cp ./builds/pygmy-darwin-arm64 /usr/local/bin/pygmy`
    * for an Linux / WSL, run `sudo cp ./builds/pygmy-linux-amd64 /usr/local/bin/pygmy`

6. Mark file permission as executable.

    `sudo chmod +x /usr/local/bin/pygmy`

    Done, run `pygmy up` as usual.

### Instructions to install Pygmy-go (Windows)

!!! warning
    DO NOT USE if using WSL, use WSL instructions instead.

1. Install choco - Open PowerShell as Admin and run:

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force
    iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
    ```

2. Install make - Run `choco install make`

3. Clone the pygmy-go repo

    Change to directory. e.g. `cd C:\programs`

    Run `git clone https://github.com/pygmystack/pygmy.git pygmy-go`

    Change to directory to pygmy-go. e.g. `cd C:\programs\pygmy-go`

4. Compile pygmy-go to make pygmy.exe

    !!! note
        * If you have a SSL scanning program, e.g. zscaler by your organization, ensure the extra ssl cert is added to docker file first to avoid error "unable to get local issuer certificate"!
        * If you get error `reading https://proxy.golang.org/golang.org/[projectfile]: 403 Forbidden`, then add `ENV GOPROXY="https://goproxy.io,direct"` prior to `WORKDIR` instruction in the `Dockerfile`. and try again a few times.

    ??? info "Instructions for Zscaler (windows)"

        1. Open Windows Start menu
        2. "mmc.exe"
        3. File
        4. add /remove snap-in
        5. Certificates
        6. Add
        7. Either My account or Computer account, doesn't matter (won't appear if not run as admin - so can be skipped)
        8. OK
        9. [In left side bar]
        10. Trusted Root Certification Authorities
        11. Certificates
        12. Find Zscaler Root Certificate
        13. Right click
        14. Tasks
        15. Export (Format CER - base64) to [pygmy folder]/zscaler.cer

        Then Add the following lines to the `[pygmy folder]/Dockerfile` after `COPY service/ /go/src/github.com/pygmystack/pygmy/service/`

        ```dockerfile
        RUN apk update
        RUN apk add curl openssl
        RUN CACHE_BUST_COUNTER=1 && echo "Cache bust counter: $CACHE_BUST_COUNTER"
        COPY zscaler.cer /tmp/
        RUN openssl x509 -inform PEM -in /tmp/zscaler.cer -out /usr/local/share/ca-certificates/zscaler.crt
        RUN update-ca-certificates
        ```

        Note: if you get error `Could not find certificate from /tmp/zscaler.cer` then your `zscaler.cer` is probably a `DER encoded binary X.509 (.CER)` but it should be a **`Format CER - base64`** format, so reexport the certificate and re-edit the Dockerfile to increase the `CACHE_BUST_COUNTER=1` so the cache get busted, thus grabs the new exported new certificate file.

        See <https://help.zscaler.com/zia/adding-custoem-certificate-application-specific-trusted-store#docker-file>

    Run `make build;`

5. Add pygmy.exe to PATH

    Run `systempropertiesadvanced.exe` => Advanced tab => Environment Variables > Click edit on PATH for System variables.
    Add builds to that PATH property e.g. `C:\programs\pygmy-go\builds`

6. Open to new command prompt (cmd.exe)

    This will have the new PATH, then run `pygmy up` as usual.

### Changing the pygmy-go ports

1. To change the pygmy-go's haproxy port (website port), add a file `.pygmy.yml` to your user folder. e.g. `C:\users\myusername\.pygmy.yml`

2. And Put the contents (adjust 7088 / 7089 to the port you want)

    ```yml
    services:
      amazeeio-haproxy:
        HostConfig:
          PortBindings:
            80/tcp:
              - HostPort: 7088
            443/tcp:
              - HostPort: 7089
    ```

3. Then clean up pygmy-go's containers and start them again.
    (ignore the error: Error response from daemon: error while removing network: network amazeeio-network id deadXXXX has active endpoints)

    ```bash
    pygmy clean
    pygmy up
    ```

    You should see the message: `Using config file: C:\Users\myusername\.pygmy.yml` when `pygmy up` is run.

    You can also see the current configuration by running `pygmy export --output /path/to/output-config-file.yml`

## Building your project

These instructions will help you set up your project locally. It assumes that you have been provisioned with a GovCMS project in [Gitlab](https://projects.govcms.gov.au). If you have not been provisioned with a project, you can **test** this process by installing the [scaffold](https://github.com/govcms/scaffold), and skipping to **step 3**.

1. Validate that you have the tools you need. Refer to [these commands](https://gist.github.com/simesy/342b6cbd3d20cd2764e17ad162d3c3cb) for help.

2. Clone your project's Gitlab repository. The clone URL can be found at `https://projects.govcms.gov.au/ORGNAME/PROJECTNAME`

    Your should ensure that the location of your clone is included in [Docker's file sharing](https://docs.docker.com/docker-for-mac/#file-sharing).

    ```bash
    git clone git@projects.govcms.gov.au:ORGNAME/PROJECTNAME.git
    cd PROJECTNAME
    ```

3. Build and start the docker containers:

    ```bash
    # This is identical to `ahoy up`.
    docker-compose up -d
    ```

4. Build the codebase with Composer:

    ```bash
    # This is identical to `ahoy composer install`.
    docker-compose exec -T test composer install
    ```

5. Install a vanilla GovCMS site:

    ```bash
    # This is identical to `ahoy install`.
    docker-compose exec -T test drush si -y govcms
    ```

6. You should have a running site now, which you can visit at `http://PROJECTNAME.docker.amazee.io`. To login to Drupal as user 1 you can run:

    ```bash
    # This is identical to `ahoy login`.
    docker-compose exec -T test drush uli
    ```
