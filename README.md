# Flow Scripts

Utility tools for Flow. Provides a few helpful functions like generating stubs and finding files that are not covered by Flow. Refer to the [commands section](#available-commands) for more.

If you have suggestions for further commands, feel free to [create an issue](https://github.com/yangshun/flow-scripts/issues/new).

## Installation

```
$ npm install -g flow-scripts
$ flow-scripts <command> [options]
```

## Available Commands

- [`stub`](#stub)
- [`unmonitored`](#unmonitored)

## `Stub`

```
$ flow-scripts stub
```

#### What it does

Generates naive stubs for the packages that your project requires. To be used with `flowignore`-ing of `node_modules` for faster start up times.

#### When should you use this?

If you are:

1. You are thoroughly annoyed by your web app project taking so long to start up because of Flow and want to `flowignore` the `node_modules` folder
1. You do not want to check in community libdefs into your repository 
1. You are okay with not having Flow libdefs for external libraries.

It also possible to combine usage of `flow-typed install` with `flow-scripts stub` as stubs for existing libdefs found in `flow-typed/npm/` will not be generated.

#### Motivation

By default, running Flow on start up will read all the files under `node_modules`. This takes very very long and it is a huge pain to be waiting that long each time your web app starts. According to Facebook, ignoring the `node_modules` directory isn't a good idea because Flow looks in there to a) ensure you've actually installed your dependencies and b) find Flow libdefs for packages which might have included them, and Flow will throw a `Required module not found` error. Refer to this [issue](https://github.com/facebook/flow/issues/869) for an in-depth discussion on the topic.

However, combined with the inclusion of libdefs (or stubs) for external libraries, ignoring `node_modules` might not seem like that bad an idea after all.

#### Workaround

The workaround is to `flowignore` the `node_modules` directory and include the libdefs inside the `flow-typed/` directory or provide a stub for it. This can be done manually or automatically via the `flow-typed install` command.

The `flow-typed install` command does fetch community libdefs and generates stubs pretty well, but has a few problems:

1. Some users might not want to fetch community libdefs because it adds many files to the source.
1. The libdefs of some libraries are not pulled into `flow-typed/npm/`, such as `immutable` because it is already present in `node_modules/immutable`. This is not picked up because we `flowignore` the `node_modules`.

The `flow-scripts stub` command fixes some of these problems by generating the stubs required for the `dependencies` in `package.json`. If there are existing libdef files in the `flow-typed/npm` directory, the stubs for these libraries will not be generated.

In `.flowconfig`, add:

```
[ignore]
.*node_modules/.*
```

In the project directory, run:

```
$ flow-scripts stub
```

This will do the following:

1. Tell Flow to ignore checking of `node_modules`.
2. Generates the stubs required for the `dependencies` in `package.json` that are not present in `flow-typed/npm/` and write them into `flow-typed/package-dep-libdefs.js`.

**Optional:** By adding the script to an npm script `postinstall` hook, when new packages are installed, it will be automatically added into `flow-typed/`. It would be recommended to save `flow-scripts` as a `devDependency` rather than a global dependency in this case.

In `package.json`, add this `postinstall` hook:

```
"scripts": {
  "postinstall": "flow-scripts stub"
},
```

#### Known Issues

- A very barebones stub of the library is being generated based solely on the package name. If you are using `lodash` and want to import a specific function like `lodash/omit`, it will not work as the stub generated by `flow-scripts stub` is for `lodash` and not `lodash/omit`. In that case, you might want to pull in a community libdef from [flow-typed](https://github.com/flowtype/flow-typed) instead.

#### TODO

- Pull out Flow libdefs for packages that already contain them such as `immutable`.

## `Unmonitored`

```
$ flow-scripts unmonitored [options] [pattern]
```

#### What it does

Searches for files matching the specified [glob pattern](https://www.wikiwand.com/en/Glob_(programming)) and lists the files that do not contain `@flow`. Files that have `@noflow` are ignored. If `pattern` is not specified, it defaults to `./**/*.{js,jsx}`. Please note that this commands works on files only, and not directories, hence you will have to explicitly specify a file extension. You will also have to quote your parameter (use double quotes if you need it to run in Windows). An example as follows:

```
$ flow-scripts unmonitored "src/**/*.js"
```

#### Options

- `--fix`: Automatically fix those files by adding `// @flow` at the top.

## Development

Testing this library is tricky because it relies on a real project that has multiple dependencies in `package.json`. Hence we create a mock project in the `fixtures` folder that has some common JS dependencies defined and symlink the `flow-scripts` library within that project to our development file in the root folder. Run the commands within that mock project to test that the library is actually working as intended.

```
$ cd test-project
$ npm install # or yarn install
$ npm link ../
$ flow-scripts stub # flow-typed/package-dep-libdefs.js file should be generated
```
