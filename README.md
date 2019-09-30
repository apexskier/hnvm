# HNVM: Hermetic Node Version Manager

Hermetically sealed versions of [node](https://npmjs.org) and [pnpm](https://pnpm.js.org).

Instead of relying on the version of `node`, `npm`, and `pnpm` each individual developer or machine
has installed locally, packages use a system we've named HNVM, which stands for Hermetic Node
Version Manager. Having our `node` binaries be hermetic means each app or package defines what
version of `node` they depend on. That version is installed and used to run whenever the package
runs scripts in `node`, `npm`, and `pnpm`. This ensures that everyone's systems output consistent
packages, and upgrades required by one package don't affect another.

## Installation

HNVM is distributed via [Homebrew](https://brew.sh) inside the
[`UrbanCompass/versions` tap](https://github.com/UrbanCompass/homebrew-versions):

```sh
brew tap UrbanCompass/versions

# If you get a permissions error tap'ing, try using the repo's ssh url
brew tap UrbanCompass/versions git@github.com:UrbanCompass/homebrew-versions.git

brew install hnvm
```

## Usage

HNVM reads the version of `node` and `pnpm` set in your `package.json` file' `"engines"` field. If
no version is set, it will default to the current versions set in HNVM's own `package.json`. Unlike
HNVM 1.0, you don't have to find any particular bash script to run HNVM. Just use the regular
`node`, `npm`, and `pnpm` commands you're used to from anywhere on your computer. If you run it
next to a `package.json` file, it will read the engines field. If not, it'll default to the global
version.

```js
// main.js
console.log('Node version', process.version);
```

```sh
# package.json in this directory has the hnvm version set to 12.10.0

node main.js # Echo's: "Node version v12.10.0
```

## Configuration

HNVM reads its configuration from one of the following places, in order from highest to lowest
priority:
1. The `package.json` file in the current process working directory (`$PWD/package.json`)
2. An `.hnvmrc` file in the `PWD` (`$PWD/.hnvmrc`)
3. An `.hnvmrc` file at the root of your git repo (if running in a git repo)
4. An `.hnvmrc` file in your home directory (`~/.hnvmrc`)
5. The `.hnvmrc` file that ships with the current version of HNVM

The `.hnvmrc` files are all simple key/value pairs prepended with `HNVM_`:
```sh
HNVM_PATH=/path/to/.hnvm
HNVM_NODE=10.0.0
HNVM_PNPM=3.0.0
```

Node and PNPM versions configured in `package.json` files go in either the `"engines"` field or an
`"hnvm"` field, with the latter overriding the former:
```js
{
  "name": "some-package",
  "version": "1.0.0",
  "engines": {
    "node": "10.0.0",
    "pnpm": "3.0.0"
  },
  "hnvm": {
    "node": "11.0.0" // This overrules any versions set in "engines"
  }
}
```

The `"hnvm"` field can contain any of the options set in the `.hnvmrc` files, without the `HNVM_`
prefix, except for the path:

```js
{
  "name": "some-package",
  "version": "1.0.0",
  },
  "hnvm": {
    "node": "11.0.0"
    "pnpm": "3.1.0"
    "quiet": true,
  }
}
```

The full list of config options are detailed below.

### `HNVM_PATH` (Defaults to `~/.hnvm`)

Location on disk to download binaries to, defaulting to an `.hnvm` directory in your `$HOME`
directory.

### `HNVM_NODE` and `HNVM_PNPM` (Defaults to `latest`)

Version of Node and PNPM to use. If semver ranges are provided instead of exact versions (e.g. the
defaults are set to `latest`), HNVM will perform curl requests to resolve those to an exact version.
However you'll get a warning about this since it could slow down execution time from the async
request, or it might even fail to work at all if the curl requests fail to load.

It's best to create an `.hnvmrc` file in your home directory and set the versions to exact versions.

### Range Cache (Defaults to 60)

To try to mitigate the slowdown above, semver range results are cached locally. The default is 60s
but you can control this by setting the `HNVM_RANGE_CACHE` environment variable. To disable the
cache set the value to 0.

### Quieting output (Defaults to `false`)

HNVM outputs information about the current version of node running, and any download statuses if a
new version is being downloaded so that you know exactly what's going on. If you don't want this
output, set the `HNVM_QUIET` environment variable to `true` to have this output redirected to
`/dev/null`.

## Why Not Just Use [NVM](https://github.com/nvm-sh/nvm)

Because `nvm` doesn't support [`fish`](https://fish.sh) 😅. Additionally, it's gotten a bit too
bloated by trying to do too many things. HNVM is focused on using and running node at a specific
version if declared, not in having multiple environments globally.
