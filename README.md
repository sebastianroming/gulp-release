# <img src="https://www.dropbox.com/s/tbm0ulwypyfcppi/gulp-gitflow.png?raw=1" width="80" height="80"> gulp-release
[![Build Status](https://travis-ci.org/nfantone/gulp-release.svg?branch=develop)](https://travis-ci.org/nfantone/gulp-release)[![js-semistandard-style](https://img.shields.io/badge/code%20style-semistandard-brightgreen.svg?style=flat-square)](https://github.com/Flet/semistandard)

A [GulpJS](https://github.com/gulpjs) plugin that enables support for git-flow style releases. It requires the [gitflow command line tool](https://github.com/petervanderdoes/gitflow-avh) to be installed on your system.

## Gitflow
A proposed workflow revolving around the use of git as a central tool. Defines a branching model to follow using best practices and convenience directives.
- See the [original concept](http://nvie.com/posts/a-successful-git-branching-model/) at nvie.com.
- Read the [derivations](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) forged by the people at Atlassian.

## Installation
`gulp-release` is a **GulpJS plugin**. It defines custom tasks that group common flow-related behaviors when releasing software using git.

```bash
npm i --save-dev gulp-release
```

### Requirements
- gitflow `^1.9.0`
- gulp `~3.9`
- node `^4.0.0`
- npm `^3.0.0`

```bash
# Add node repositories
curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -

# Install dependencies
sudo apt-get install git-flow
sudo apt-get install -y nodejs
sudo npm i -g npm
sudo npm i -g gulp
```

## Usage
### Simple
In your `gulpfile.js` declare:

```javascript
'use strict';
const gulp = require('gulp');
const release = require('gulp-release');

release.register(gulp);
```

This will register tasks on your `gulp` instance. After that, you'll be able to call (or depend upon) all the tasks described below, such as:

```bash
$ gulp release
```

### Advanced
You may pass in an `options` configuration object to the `register` method to override some or all of the default settings.

```javascript
'use strict';
const gulp = require('gulp');
const release = require('gulp-release');

release.register(gulp, { packages: ['package.json'] });
```

Here, the `packages` property expects an ordered array of _globs_ that match `json` files containing version information (i.e: `package.json`, `bower.json`, `component.json`). All version files will be updated on a release, but version number will be read _only_ from the first one.

> Read next section for a complete set of available options.

## API
### `register(gulpInst[, options])`
Declares tasks on the `gulpInst` gulp instance. `options` is an optional object that can be defined to override the following sensible defaults:

```javascript
 {
     tasks: {
        // Name of the main task (a.k.a, `gulp release`)
       release: 'release'
     },
     messages: {
        // Commmit message on version bumping
       bump: 'Bump release version',

       // Commit message on setting next "-dev" version on `develop`
       next: 'Set next development version'
    },

    // Supports glob syntax
    packages: ['package.json'],

    // Suffix added on development versions (ie.: on `develop` branch)
    devSuffix: '-dev',

    // Path to a custom codenames file. See https://github.com/scriptwerx/gulp-codename/blob/master/codenames.json
    codenames: 'my-codenames.json' 
};
```

This parameters permits you to configure main task name, commit messages and `.json` files containing a `.version` attribute that will be bumped on a new release.

## Tasks
### `release`
Performs a full, automatic project release. Uses `git flow release` internally. Name is configurable.

> It's required that your version numbering follows [semver](http://semver.org/) specifications.

```bash
gulp release [-v --version VERSION] [-t --type TYPE] [-m CODENAME] [-c --codenames PATH] [-p --push]
```

The repository you invoke this task on, must be `git flow` enabled. Run `git flow init` if you haven't already, before running `gulp release`. Otherwise, the task will fail.

- `-v` or `--version` can be used to indicate a specific **next** version (such as `3.2.2-beta`). If unspecified, the version from your first configured package file (ie., `package.json`) will be used.
- `-t` or `--type` can be used to indicate types of version increment (_MAJOR_, _MINOR_ or _PATCH_). Next release version defaults to _PATCH_ increment.
- If your current version ends with a suffix, next default version will be that same number without the suffix (`0.0.2-dev` -> `0.0.2`).
- `-m` provides a human readable "codename" for the release. One will be autogenerated if not provided.
- `-c` or `--codenames` provides a path to a [custom codenames JSON file](https://github.com/scriptwerx/gulp-codename#custom-options). This takes precedence over the `.register()` config option if both are set.
- `-p` or `--push` indicate whether to push results (branches and tags) to `origin` or not after finishing the release process. Defaults to `false`.

This recipe will perform the following actions, sequentially (default option values are assumed):

1. Invoke `git flow release start -F <version>`. Where `<version>` is either set from a command line argument or read from a package file.
2. Bump the version on all package files and [generate a codename](https://www.npmjs.com/package/gulp-codename) for the release if one was not already provided with `-m`.
3. Commit changes from last step to `develop` using `"Bump release version"` as message.
4. Invoke `git flow release finish -m <codename> <version>`.
5. Bump the version on all packages files to _next development iteration_ using a `-dev` suffix (like `1.0.1-dev`).
6. Commit changes from last step to `develop` using `"Set next development version"` as message.
7. If the `-p` (or `--push`) flag was set, push all tags and local `develop` and `master` branches to `origin`.


> See the [git flow release wiki](https://github.com/petervanderdoes/gitflow-avh/wiki/Reference:-git-flow-release#reference----git-flow-release) for details on what's happening under the hood when calling `git flow release start|finish`.

## License
MIT
