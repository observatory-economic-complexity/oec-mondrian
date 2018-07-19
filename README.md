# OEC Mondrian

## Getting started

1. Install [JRuby](http://jruby.org). Use of [RVM](https://rvm.io/) is recommended.
2. Run `gem install bundler` if bundler is not installed yet.
2. Install dependencies with `bundle install`
3. Add the appropriate environment variables to adjust DB connection parameters in `config.ru` (example below):

```
export DB_URL=jdbc:monetdb://localhost:50000/oec
export DB_NAME=oec
export DB_USER=monetdb
export DB_PASSWORD=monetdb
```

4. Run server with `JRUBY_OPTS=-G rackup` or `just serve` (see details below)

## Writing schemas

You have two options:
1) Directly write your schema into schema.xml in the root of the repo.
2) Write your schema in fragments, and have them concatenated by a utility to the schema.xml.

There's no special workflow around writing the schema for choice 1.

If you use choice 2 (convenient for very large schemas), the solution at this moment is to put all schema frags into the `frags` directory and run [moncat](https://github.com/hwchen/mondrian-schema-cat) on the directory. An example command is found in the [task runner justfile](justfile).

## Workflow - utilities

[@hwchen](https://github.com/hwchen/) has used and written a few utilities to make developing easier, especially from the command line.

Note that all of the utilities listed use Rust, but some can be installed by `brew` or easily built directly. If you prefer other utilities, I've described the basic function for each one so you should be able to swap in your favorite (for non-mondrian specific).

- [just](https://github.com/casey/just), a task-runner. Homebrew and prebuilt binaries install.
- [watchexec](https://github.com/mattgreen/watchexec), watch for file changes and execute command. Homebrew and prebuilt binaries install.
- [moncat](https://github.com/hwchen/mondrian-schema-cat), intelligently concat mondrian schema frags, clone repo and build from source (See Rust Section below).
- [moncli](https://github.com/hwchen/mondrian-rest-cli), cli interface to mondrian-rest, clone repo and build from source (See Rust Section below).
- `curl` should already be installed on your computer, unless you use windows.

(For `moncat` and `moncli`, @hwchen may have prebuilt binaries available if requested).

## Workflow - flow for local mondrian server

Ensure your system already has [just](https://github.com/casey/just) installed.

The justfile has the main setup tasks.
- `just serve` starts up the server with good config. Necessary for correct dimension naming conventions and to get sql query debugging output.
- `just cat` sets up a watcher that will automatically update the schema.xml whenever you update the frags dir.

Then, the general idea is to:
- make a change in the schema
- `just flush` to flush the mondrian server cache and reload schema (or curl or use moncli to same effect)
- `moncli -b http://localhost:9292 <describe|query|etc>` to inspect output

`moncli` here is an alias for `mondrian-rest-cli`. More information on query options can be find in the [readme](https://github.com/hwchen/mondrian-rest-cli). Don't forget to use the `--format csv` flag for easy-on-the-eyes output.

Another good option for inspecting the output easily is to use [mondrian-rest-ui](https://github.com/Datawheel/mondrian-rest-ui); it takes a little more time to set up its node server though.


(Note that changes to configuration of the mondrian server will require a restart).

(This can be adapted to a remote mondrian server, just change the mondrian base url).

## UI
Use this [mondrian-rest-ui](https://github.com/Datawheel/mondrian-rest-ui) to inspect data using a web app. Requires running a node server.

## Rust section

Get rust [here](https://www.rust-lang.org/en-US/install.html).

Then you should be able to do `cargo install` in the cloned repo of a utility to install.

Or `cargo install <package-name>` if you know the package name and it's published to [crates registry](https://crates.io/)
