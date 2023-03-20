# Neko Scalingo Buildpack

This repository is a [custom buildpack](https://doc.scalingo.com/platform/deployment/buildpacks/custom) to build Haxe on Scalingo.

## Introduction on buildpacks

A buildpack has three mandatory entrypoints:

- bin/detect: exit with success (return code is 0) if the buildpack applies to the current application.
- bin/compile: installs the dependencies of the project. It is called with three arguments:
  - The build directory: contains the code of the application. _BUILD_DIR=${1:-}_
  - The cache directory: used to store information one want to keep between two builds. _CACHE_DIR=${2:-}_
  - The environment directory: contains a file per environment variable defined. For instance, an environment variable TEST=1234 leads to a file named TEST containing 1234. _ENV_DIR=${3:-}_
- bin/release: handles some metadata and defines how the application should be started.

All these entrypoints are usually Bash script.

## Multi buildpacks and APT Buildpack

This buildpack will only works in a [Multi Buildpacks](https://doc.scalingo.com/platform/deployment/buildpacks/multi) with the [APT Buildpack](https://doc.scalingo.com/platform/deployment/buildpacks/apt)

The APT buildpack is used to install additional packages using the `apt` package manager (i.e. node, npm, etc). The packages to install needs to be in a `Aptfile` file at the root of your project.
The Aptfile needed for the Neko Scalingo Buildpack can be found in this repo.

The Multi Buildpacks buildpack is used to have multiple buildpacks in your application. To make the Neko Scalingo Buildpack works, you need to have a `.buildpacks` file at the root of your project which defines the buildpacks to use.
The .buildpacks needed for the Neko Scalingo Buildpack can be found in this repo.

## compile script

The compile script will install lix, then the Haxe dependancies. Then it will build the backend and the frontend of your project. It will also generate mtt files.
Finally, it will create a `.profile.d` script. This script will be executed by Scalingo during the startup of a container. This script will run the `update-config.pl` Perl script that will create the `config.xml` from some runtime environement variables.

## release script

The release script prints on the standard output a YAML file with a `default_process_types` key which contains the default [Procfile](https://doc.scalingo.com/platform/app/procfile) entry. This entry will launch a Scalingo `web` container with an Apache server using the Apache config located in `/app/apache-scalingo.conf`.

## Test build with:

```shell :eval,no
  docker run --pull always --rm -ti -e STACK=scalingo-20 \
         -v $PWD:/buildpack \
         -v $PWD/../app/:/build scalingo/scalingo-20:latest bash
```

## Buildpack github token

This custom buildpack is stored in a private repo. The buildpack URL must include a valid github token (aka `PAT`).

    BUILDPACK_URL=https://<token>@github.com/CAMAP-APP/neko-scalingo-buildpack-camap.git#main

# DB

DB firewall was updated to allow access from [Scalingo's egress address](https://doc.scalingo.com/platform/internals/network#osc-fr1-region)(`171.33.105.206`)
