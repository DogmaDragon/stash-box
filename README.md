# stash-box

[![Discord](https://img.shields.io/discord/559159668438728723.svg?logo=discord)](https://discord.gg/2TsNFKt)

**stash-box is Stash App's own OpenSource video indexing and Perceptual Hashing MetaData API for porn.**

The intent of stash-box is to provide a collaborative, crowd-sourced database of porn metadata, in the same way as [MusicBrainz](https://musicbrainz.org/) does for music. The submission and editing of metadata is expected to follow the same principle as that of the MusicBrainz database. [See here](https://musicbrainz.org/doc/Editing_FAQ) for how MusicBrainz does it.

Currently, stash-box provides a graphql backend API only. There is no built in UI. The graphql playground can be accessed at `host:port/playground`. The graphql interface is at `host:port/graphql`.

# Docker install

TODO

# Bare-metal Install

Stash-box supports macOS, Windows, and Linux.  

Releases TODO

## Initial setup

Stash-box requires access to a postgres database server. When stash-box is first run, or when it cannot find a configuration file (defaulting to `stashdb-config.yml` in the current working directory), then it generates a new configuration file with a default postgres connection string (`postgres@localhost/stash-box?sslmode=disable`). It prints a message indicating that the configuration file is generated, and allows you to adjust the default connection string as needed.

The database must be created and available before rerunning stash-box. The schema will be created within the database if it is not already present.

The `sslmode` parameter is documented in the [pq documentation](https://godoc.org/github.com/lib/pq). Use `sslmode=disable` to not use SSL for the database connection. The default value is `require`.

After ensuring the database connection string is correct and the database server is available, the stash-box executable may be rerun. 

The second time that stash-box is run, stash-box will run the schema migrations to create the required tables. It will also generate a `root` user with a random password and an API key. These credentials are printed once to stdout and are not logged. The root user will be regenerated on startup if it does not exist, so a new root user may be created by deleting the root user row from the database and restarting stash-box.

## CLI

Stash-box provides some command line options.  See what is currently available by running `stashdb --help`.

For example, to run stash locally on port 80 run it like this (OSX / Linux) `stashdb --host 127.0.0.1 --port 80`

## Configuration

Stash-box generates a configuration file `stashdb-config.yml` in the current working directory when it is first started up. This configuration file is generated with the following defaults:
- running on `0.0.0.0` port `9998`

### API keys and authorisation

A user may be authenticated in one of two ways. Session-based management is possible by logging in via `/login`, passing form values for `username` and `password` in plain text. This sets a cookie which is required for subsequent requests. The session can be ended with a request to `/logout`.

The alternative is to use the user's api key. For this, the `ApiKey` header must be set to the user's api key value.

## SSL (HTTPS)

Stash-box supports HTTPS with some additional work.  First you must generate a SSL certificate and key combo.  Here is an example using openssl:

`openssl req -x509 -newkey rsa:4096 -sha256 -days 7300 -nodes -keyout stashdb.key -out stashdb.crt -extensions san -config <(echo "[req]"; echo distinguished_name=req; echo "[san]"; echo subjectAltName=DNS:stashdb.server,IP:127.0.0.1) -subj /CN=stashdb.server`

This command would need customizing for your environment.  [This link](https://stackoverflow.com/questions/10175812/how-to-create-a-self-signed-certificate-with-openssl) might be useful.

Once you have a certificate and key file name them `stashdb.crt` and `stashdb.key` and place them in the directory where stash-box is run from. Stash-box detects these and starts up using HTTPS rather than HTTP.

# FAQ

> I have a question not answered here.

Join the [Discord server](https://discord.gg/2TsNFKt).

# Development

## Install

* [Revive](https://github.com/mgechev/revive) - Configurable linter
    * Go Install: `go get github.com/mgechev/revive`
* [Packr2](https://github.com/gobuffalo/packr/tree/v2.0.2/v2) - Static asset bundler
    * Go Install: `go get github.com/gobuffalo/packr/v2/packr2@v2.0.2`
    * [Binary Download](https://github.com/gobuffalo/packr/releases)
* [Yarn](https://yarnpkg.com/en/docs/install) - Yarn package manager

NOTE: You may need to run the `go get` commands outside the project directory to avoid modifying the projects module file.

## Environment

### macOS

TODO

### Windows

1. Download and install [Go for Windows](https://golang.org/dl/)
2. Download and install [MingW](https://sourceforge.net/projects/mingw-w64/)
3. Search for "advanced system settings" and open the system properties dialog.
    1. Click the `Environment Variables` button
    2. Add `GO111MODULE=on`
    3. Under system variables find the `Path`.  Edit and add `C:\Program Files\mingw-w64\*\mingw64\bin` (replace * with the correct path).

## Commands

* `make generate` - Generate Go GraphQL and packr2 files. This should be run if the graphql schema or schema migration files have changed.
* `make build` - Builds the binary
* `make vet` - Run `go vet`
* `make lint` - Run the linter
* `make test` - Runs the unit tests
* `make it` - Runs the unit and integration tests

**Note:** the integration tests run against a temporary sqlite3 database by default. They can be run against a postgres server by setting the environment variable `POSTGRES_DB` to the postgres connection string. For example: `postgres@localhost/stash-box-test?sslmode=disable`. **Be aware that the integration tests drop all tables before and after the tests.**

## Building a release

1. Run `make generate` to create generated files 
2. Run `make build` to build the executable for your current platform

## Cross compiling

TODO
