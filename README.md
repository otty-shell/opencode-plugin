# otty-plugin-opencode

The official OpenCode plugin of OTTY. OTTY has no code for any agent by
name: this plugin teaches it OpenCode. Its one agent part, `opencode`,
finds OpenCode's data folder (`~/.local/share/opencode`, or the one
`XDG_DATA_HOME` names) and reads the sessions and the tokens and the
cost of every answer, with its session and project, from the database
(`opencode.db`, or the one `OPENCODE_DB` names). The cost OpenCode wrote
is given as it is, so the money is exact; an answer with no cost is
priced at the API price OTTY gives for its model.

It is an agent only: it has no provider part and no login code.
OpenCode works with no login, with its free models, so the part never
says a login is missing and asks the user for nothing. The logins
OpenCode keeps, in `auth.json` and in the `credential` table, are never
read; the manifest still declares them as keys, so OTTY never gives
them to the part as text.

OTTY opens the database read-only, so the plugin never writes to it.

OpenCode's installer puts the program at `~/.opencode/bin/opencode`, a
folder the login shell may not have on its `PATH`: the manifest lists it
in `program_places`, so OTTY finds the program there and starts it by
that path. When OpenCode is nowhere, the manifest's `install` gives
OpenCode's install script, `curl -fsSL https://opencode.ai/v2/install |
bash`, which OTTY runs in a terminal tab for **Install OpenCode**, and
OpenCode's page, `https://opencode.ai/docs`.

## Build

The plugin takes `otty-plugin-sdk` from the `agents` branch of the OTTY
repository (`Cargo.toml`). The first build pins the SDK in `Cargo.lock`
to the newest commit of that branch; commit the lock, and run `cargo
update -p otty-plugin-sdk` to take a newer SDK. Its tests run natively
over made-up data folders: the files of `fixtures/`, which stand for the
home folder, and databases the tests write in a temporary folder.
`fixtures/.local/share/opencode/opencode.db` is made from
`fixture-database.sql` (the command is at its top).:

```sh
rustup target add wasm32-wasip2
cargo test
cargo build --release --target wasm32-wasip2
```

The component is `target/wasm32-wasip2/release/otty_plugin_opencode.wasm`.

To build against an OTTY checkout instead, such as one next to this
folder, point cargo at its SDK for one command, without changing
`Cargo.toml`:

```sh
cargo --config 'patch."https://github.com/otty-shell/otty".otty-plugin-sdk.path="../otty/crates/plugins/sdk"' test
```

## Install

In OTTY, open Plugins mode, press **Add plugin…**, paste
`https://github.com/otty-shell/otty-plugin-opencode`, then **Next** and
**Install**. OTTY installs the newest release it can run and offers the
later ones as updates.

To try a build that is not released, put the component as
`opencode.wasm` in a folder with `otty-plugin.toml` and the icons
(`opencode.svg`), and install that folder with **Add plugin from
folder…**. After a change, copy the new build there and press **Reload**
on the plugin's row.

## Release

Set the same `version` in `otty-plugin.toml` and `Cargo.toml`, commit,
and push the tag `v<version>`:

```sh
git tag v0.1.12
git push origin v0.1.12
```

The workflow `.github/workflows/release.yml` checks that the tag equals
`version` of `otty-plugin.toml`, runs the tests, builds the plugin for
`wasm32-wasip2` and attaches to the GitHub release of the tag the files
OTTY installs: `otty-plugin.toml`, `opencode.wasm` and the icons. OTTY
refuses a release whose manifest names another version than its tag.
