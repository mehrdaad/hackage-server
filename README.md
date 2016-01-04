# hackage-server

[![Build Status](https://travis-ci.org/haskell/hackage-server.png?branch=master)](https://travis-ci.org/haskell/hackage-server)

This is the `hackage-server` code. This is what powers <http://hackage.haskell.org>, and many other private hackage instances.

## Installing ICU

ICU stands for "International Components for Unicode". The `icu4c` is a set
of libraries that provide Unicode and Globalization support.
The [text-icu](https://hackage.haskell.org/package/text-icu) Haskell package
uses the [icu4c](http://icu-project.org/apiref/icu4c/) library to build.

You'll need to do the following to get hackage-server's dependency `text-icu` to build:

### Mac OS X

    brew install icu4c
    brew link icu4c --force

### Ubuntu/Debian

    sudo apt-get update
    sudo apt-get install unzip libicu-dev

## Running

    cabal install -j --enable-tests

    hackage-server init
    hackage-server run

If you want to run the server directly from the build tree, run

    dist/build/hackage-server/hackage-server run --static-dir=datafiles/

By default the server runs on port `8080` with the following settings:

    URL:      http://localhost:8080/
    username: admin
    password: admin

To specify something different, see `hackage-server init --help` for details.

The server can be stopped by using `Control-C`.

This will save the current state and shutdown cleanly. Running again
will resume with the same state.

### Resetting

To reset everything, kill the server and delete the server state:

```bash
rm -rf state/
```

Note that the `datafiles/` and `state/` directories differ:
`datafiles` is for static html, templates and other files.
The `state` directory holds the database (using `acid-state`
and a separate blob store).

### Creating users & uploading packages

* Admin front-end: <http://localhost:8080/admin>
* List of users: <http://localhost:8080/users/>
* Register new users: <http://localhost:8080/users/register>

Currently there is no restriction on registering, but only an admin
user can grant privileges to registered users e.g. by adding them to
other groups. In particular there are groups:

 * admins `http://localhost:8080/users/admins/` -- administrators can
   do things with user accounts like disabling, deleting, changing
   other groups etc.
 * trustees `http://localhost:8080/packages/trustees/` -- trustees can
   do janitorial work on all packages
 * mirrors `http://localhost:8080/packages/mirrorers/` -- for special
   mirroring clients that are trusted to upload packages
 * per-package maintainer groups
   `http://localhost:8080/package/foo/maintainers` -- users allowed to
   upload packages
 * uploaders `http://localhost:8080/packages/uploaders/` -- for
   uploading new packages

### Mirroring

There is a client program included in the hackage-server package called
hackage-mirror. It's intended to run against two servers, syncing all the
packages from one to the other, e.g. getting all the packages from the old
hackage and uploading them to a local instance of a hackage-server.

To try it out:

1. On the target server, add a user to the mirrorers group via
   http://localhost:8080/packages/mirrorers/
1. Create a config file that contains the source and target
   servers. Assuming you are cloning the packages on
   <http://hackage.haskell.org> locally, create the file servers.cfg:
```
source "hackage"
  uri: http://hackage.haskell.org
  type: hackage2

target "mirror"
  uri: http://admin:admin@localhost:8080/
  type: local

  post-mirror-hook: "shell command to execute"
```
Recognized types are hackage2, secure and local. The target server name was displayed when you ran
```bash 
   hackage-server run.
```

1. Run the client, pointing to the config file:

```bash
hackage-mirror servers.cfg
```

This will do a one-time sync, and will bail out at the first sign of
trouble. You can also do more robust and continuous mirroring. Use the
flag `--continuous`. It will sync every 30 minutes (configurable with
`--interval`). In this mode it carries on even when some packages
cannot be mirrored for some reason and remembers them so it doesn't
try them again and again. You can force it to try again by deleting
the state files it mentions.
